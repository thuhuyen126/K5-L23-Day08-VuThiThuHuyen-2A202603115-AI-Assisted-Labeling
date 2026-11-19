# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 340; đối tượng sau duyệt: 541; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 140 | 140/340 = 41.2% | 340 box máy |
| edited (sửa) | 136 | 136/340 = 40.0% | 340 box máy |
| deleted (xoá) | 62 | 62/340 = 18.2% | 340 box máy |
| added (vẽ thêm) | 263 | 263/541 = 48.6% | 541 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 255.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 76 | 72 | 14 | 131 |
| bicycle | 2 | 3 | 5 | 6 |
| motorbike | 22 | 46 | 14 | 75 |
| car | 28 | 5 | 6 | 37 |
| bus | 1 | 0 | 1 | 2 |
| truck | 1 | 1 | 0 | 7 |
| three_wheeler | 10 | 9 | 22 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 75 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 35 | 5 | 40 | 87.5% (KTC 95%: 74–95%) | 46.7% (KTC 95%: 36–58%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - missing: 3/3
  - loose: 3/3
  - extra: 0/3
  - lọt: img_003.jpg (extra), img_018.jpg (extra), img_033.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
