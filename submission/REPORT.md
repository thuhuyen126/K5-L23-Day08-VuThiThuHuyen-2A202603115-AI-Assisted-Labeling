# Báo cáo Day 08 — 2A202603115 - Vũ Thị Thu Huyền

> Bản tổng hợp theo các file và lần chạy hiện có. Những nội dung chưa có bằng chứng được ghi rõ là chưa ghi nhận, không suy đoán số liệu.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Nhận xét |
|---|---|---:|---|
| person / motorbike / car / bus / truck | YOLO26m | mặc định 0.3 | Giữ làm nguồn pre-label cho các lớp COCO; vẫn phải rà soát bỏ sót và sai lớp. |
| bicycle | YOLO26m | 0.2 | Threshold sweep gợi ý ngưỡng này trên 6 vật gold; mẫu nhỏ nên chỉ là ước lượng. |
| three_wheeler | YOLOE-26s; prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | Trên `img_002`, YOLOE nhận tuk-tuk xanh là `three_wheeler` (conf 0.93); vẫn cần review box. |

- Chuỗi ngưỡng: `--thr "bicycle=0.2,three_wheeler=0.15"`; các lớp còn lại dùng mặc định 0.3.
- Lệnh gộp `merge_prelabels.py`: chưa lưu nguyên văn trong các artifact hiện có.
- SAM đã dùng: SAM 2.1 tiny (`sam2.1_t.pt`), CPU. Run log ghi nhận 2.12 giây/ảnh ở lần chạy box-to-mask trên 2 ảnh.
- Lỗi không sửa được bằng đổi ngưỡng: ở `img_002`, YOLO26 gọi tuk-tuk là `truck` (conf 0.37). Tăng ngưỡng lên 0.5 làm box đó biến mất, không đổi nó thành `three_wheeler`; cần phân loại lại bằng người hoặc dùng model open-vocabulary như YOLOE.
- So sánh điểm: YOLO26 có box sai lớp ở `img_002` dù có điểm; YOLOE nhận đúng tuk-tuk với conf 0.93 nhưng box vẫn cần người kiểm.
- Giới hạn: threshold sweep dùng 4 ảnh gold; chọn và đánh giá trên cùng tập nên precision/recall lạc quan và mẫu nhỏ.

## 2. AI error profile

Theo `submission/qc/review_stats.md`; script loại 30 đối tượng quanh bẫy khỏi profile:

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 140 | 41.2% | 340 box máy |
| edited | 136 | 40.0% | 340 box máy |
| deleted | 62 | 18.2% | 340 box máy |
| added | 263 | 48.6% | 541 đối tượng sau duyệt |

Có 8 trường hợp đổi lớp trong deleted/added; ước tính khoảng 255 đối tượng máy bỏ sót thật. Theo số đếm lớp, `person` có nhiều đối tượng added nhất (131), tiếp theo là `motorbike` (75); `three_wheeler` có 22 box máy bị xóa.

So với gold: 35 TP, 5 FP, 40 FN trên 75 đối tượng gold; precision 87.5% (KTC 95%: 74–95%), recall 46.7% (KTC 95%: 36–58%). Kết quả chỉ từ 4 ảnh và ngưỡng được chọn trên chính tập này.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Lượt review đo được: 9/12 = 75% (KTC 95%: 47–91%), dưới mốc mục tiêu 80%.
- Bắt được: wrong_class 3/3, missing 3/3, loose 3/3; không bắt được extra 0/3. Các ảnh có bẫy extra lọt: `img_003`, `img_018`, `img_033`.
- Điều này cho thấy giữ/sửa/xóa box máy chưa đủ; cần quét toàn ảnh để tìm box thừa và không dựa vào cảm giác “box trông hợp lý”. Nguyên nhân cá nhân cụ thể cần được học viên tự ghi lại; không thể suy ra chỉ từ số liệu.
- Accepted 41.2% cùng với bắt bẫy 75% không chứng minh độ tin cậy cao: số đo cho thấy còn nhiều box phải sửa/xóa và có lỗi thừa lọt.

## 4. Máy có đáng dùng không?

- `time_log.csv` ghi gold: 4 ảnh trong 30 phút, tức 7.5 phút/ảnh. Thời gian `batch_review` chưa được ghi nên chưa thể kết luận có tiết kiệm thời gian hay không.
- Với 41.2% box máy được giữ nguyên và 48.6% đối tượng sau duyệt là added, pre-label chỉ nên dùng như gợi ý, không thay thế việc quét ảnh và review. YOLOE phù hợp hơn cho `three_wheeler` trong ví dụ đã quan sát; YOLO26 vẫn hữu ích cho lớp COCO nhưng có lỗi phân loại.
- Quy tắc giảm thiên kiến cho lần review sau:
  1. Ẩn nhãn máy và quét ảnh trước trên đủ 24/24 ảnh.
  2. Với từng ảnh, lần lượt kiểm bỏ sót, box thừa, sai lớp, rồi độ khít; không bỏ qua box vì conf cao.
  3. Nghỉ sau khoảng 10 ảnh và ghi thời gian thực vào `time_log.csv`.

## 5. Mask SAM cho 3 vật

| Ảnh | SAM | Số click | Quan sát mask | File |
|---|---|---:|---|---|
| `img_013` | SAM 2.1 tiny | 3 (2 dương, 1 âm) | 39.4% ảnh, 1 polygon; mask lấy mái/thân xe nhưng lấn vào một phần người bên trong. | `masks/tuktuk_img013.jpg` |
| `img_004` | SAM 2.1 tiny | 4 (2 dương, 2 âm) | 5.8% ảnh, 1 polygon; chủ yếu bám tuk-tuk xanh, ít tràn sang người và xe bên cạnh. | `masks/green_tuktuk_img004.jpg` |
| `img_009` | SAM 2.1 tiny | 4 (2 dương, 2 âm) | 2.2% ảnh, 1 polygon; lấy phần trước/thân xe nhìn thấy, không bao hết phần xe bị hàng hóa che. | `masks/loaded_motorbike_img009.jpg` |

Các kết quả cho thấy click âm giúp loại vùng lân cận, nhưng điểm dương và âm vẫn cần đặt theo từng vật. Đối chiếu mục 4.2 của guideline: mask xe cần tách khỏi người; phần bị che không nên được đoán. Cách xử lý hàng hóa gắn/chở trên xe cần được chốt rõ hơn trong calibration.

## 6. Calibration

Đã ghi 3 quyết định trong `submission/qc/calibration_decisions.csv`: CAL-01 về YOLO26 gọi tuk-tuk thành truck; CAL-02 về mask xe lấn vào hành khách; CAL-03 về xe máy bị hàng hóa che khuất. CAL-01 là lỗi máy quan trọng nhất trong các ví dụ đã ghi.

Tại thời điểm lập bản này, `guideline_v1.1_calibrated.md` vẫn còn nội dung mẫu, chưa phản ánh các quyết định trên. Cần cập nhật guideline và kiểm tra từng quy tắc khớp với `case_id` trước khi nộp.

## 7. QC và quyết định

Chưa có `qc_sample.csv`, `qc_review.csv` hoặc `qc_report_auto.md`; vì vậy chưa có tỷ lệ lỗi theo ảnh/đối tượng hay cận trên 95% để kết luận Release. Script review chỉ đo các thống kê pre-label/gold/bẫy; chưa thay thế QC thủ công.

**Quyết định hiện tại: Hold — 2026-11-19.** Lý do: chưa chạy QC và tỷ lệ bắt bẫy lượt review là 9/12 (75%), dưới mốc 80%. Chưa đủ bằng chứng để Release. Cần hoàn tất QC, xử lý các cảnh báo box nhỏ theo guideline, rồi ra quyết định cuối dựa trên số liệu thật.

## 8. Ghi chú cá nhân

- Có dùng Copilot/AI assistant để đọc cấu trúc dự án, giải thích lỗi môi trường và hỗ trợ các lệnh. Kết quả nhãn được kiểm tra bằng script của dự án và thao tác trong CVAT.
- Việc gold có bị ảnh hưởng bởi pre-label trước khi gán xong chưa được ghi nhận; học viên cần xác nhận trung thực.
- Môi trường: Windows, chạy CPU (`torch 2.15.0+cpu`, `ultralytics 8.4.215`); dùng CVAT local qua Docker. Phiên bản CVAT chưa được ghi lại.
- Log có các lần chạy model trên máy; chưa có ghi chú xác nhận việc dùng hay không dùng cache.
- Vướng mắc môi trường: `.venv` ban đầu chỉ có các gói nhẹ nên thiếu `ultralytics`; đã cài phụ thuộc từ `requirements.txt` và xác nhận import thành công.
- Việc cần hoàn tất trước báo cáo cuối: ghi thời gian `batch_review`, cập nhật guideline v1.1, đồng bộ đúng export mới vào `submission/labels/batch_reviewed.json`, chạy QC và cập nhật mục 7 cùng quyết định cuối.
