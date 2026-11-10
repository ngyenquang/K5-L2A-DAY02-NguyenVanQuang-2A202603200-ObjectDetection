# Báo cáo cá nhân — Ngày 2 Object Detection


## 1. Thông tin học viên

- Họ tên: Nguyễn Văn Quang
- MSSV: 2A202603200
- Email: nguyenvanquang30112k4@gmail.com
- Lớp: 2A-LAB-C304
- Link repo bài làm: https://github.com/ngyenquang/K5-L2A-DAY02-NguyenVanQuang-2A202603200-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1YGHyOCbtCOShP9FnXm2wGfEZY-sTv_Uk?usp=sharing

## 2. Schema và mini guideline

Chi tiết trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema: `car=0, truck=1, bus=2, van=3`; mỗi xe một box ôm sát phần nhìn thấy; không gán xe máy, xe ba bánh, người, biển báo.

Ba ca mơ hồ chính:

1. **`drive_019` — xe cứu hộ công an** (cabin lớn, có cần cẩu): gán `truck`, vì là xe chuyên dụng thân dài gần xe tải hơn xe con.
2. **`drive_024` — xe ba bánh chở người**: không gán, vì không thuộc bốn lớp và cùng nhóm với xe máy.
3. **`drive_028`, `drive_044`, `drive_014` — rất nhiều xe nhỏ ở xa**: bản đầu vẽ hết, sau khi chấm gold đã bỏ các xe quá nhỏ (xem mục 4).

Phần cần giảng viên làm rõ: ranh giới `van`/`bus` và việc xe chuyên dụng (xe cứu hộ) thuộc `truck` hay `car`.

## 3. Annotation và self-QC

- Số file nhãn train: 50 (`drive_001.txt` đến `drive_050.txt`), không có file rỗng.
- Kết quả checker và lỗi đã xử lý: `python tools/check_yolo_labels.py` báo "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ". Không có lỗi cấu trúc cần sửa.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.txt` đến `drive_050.txt`
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: lỗi lớn nhất là vẽ **quá nhiều box xe nhỏ ở xa**. Lỗi này tôi chỉ nhận ra sau khi chấm với gold (mục 4), không phát hiện được ở bước self-QC ban đầu. Cách sửa: xóa các box xe nhỏ trong CVAT rồi export lại.
- Commit khóa nhãn ban đầu: `bea2eec` ("Initial labels (before rework)"). Lưu ý: commit này được tạo **sau khi đã nhận gold**, nhưng nhãn bên trong chưa sửa gì so với bản export đầu tiên.

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 475 | 472 |
| FP | 848 | 606 |
| FN | 178 | 181 |
| Precision | 0.359 | 0.4378 |
| Recall | 0.7274 | 0.7228 |
| F1 | 0.4808 | 0.5453 |
| mean_iou_of_matches | 0.7732 | 0.7745 |
| passed | false | false |

Kết quả cuối nằm trong [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json). Có một lần chạy trung gian với F1 0.5163 (FP 715), không nộp.

**Ngưỡng:** README.md ghi F1 ≥ 0.80, nhưng lab coach đã thông báo F1 > 0.5 là đạt. [tối 10/10] Vì vậy `passed` trong JSON vẫn là `false` theo ngưỡng cố định trong script, còn kết quả cuối F1 0.5453 vượt ngưỡng 0.5 của lab coach. Mean IoU 0.7745 đạt ngưỡng 0.70.

**Rework:** commit `7a3c219` ("Reworked labels"). Số box giảm từ 1323 xuống 1078 (bỏ 245 box). Phần lớn là `car`: car 1133 → 890, bus 108 → 109, van 62 → 59, truck 20 → 20. Box bị bỏ chủ yếu là xe nhỏ ở xa: trong 250 dòng nhãn bị bỏ, cạnh dài trung vị khoảng 19 px, 75% không quá 28 px, còn box giữ lại có cạnh dài trung vị khoảng 49 px. Các ảnh giảm nhiều box nhất: drive_014 (58 → 32), drive_044 (42 → 24), drive_007 (36 → 18), drive_049 (29 → 15), drive_028 (24 → 14).

**Còn tồn tại:** vẫn dư 606 FP. Số box của tôi (1078) còn nhiều hơn gold (653), nên có thể vẫn còn xe nhỏ chưa bỏ. Tôi chưa kiểm từng box còn lại, nên chưa kết luận chắc nguyên nhân. TP giảm 3 và FN tăng 3, tức là có vài box đúng bị bỏ nhầm khi dọn. 

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU Tesla T4; Ultralytics 8.4.208; Python 3.13.16; torch 2.14.1+cu130.
- Các tham số thay đổi so với notebook nếu có và lý do: không thay đổi (50 epoch, imgsz 640, batch tự động chọn 44, seed 20260909, checkpoint `yolo26n.pt`). Train 50 epoch mất khoảng 1.5 phút.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4105 |
| Recall | 0.2303 |
| mAP50 | 0.2723 |
| mAP50–95 | 0.2017 |

Theo từng lớp (10 ảnh eval): car 94 xe (P 0.833, R 0.637, mAP50 0.740); truck 5 xe (R 0.200, mAP50 0.239); bus 2 xe (R 0, mAP50 0.008); van 12 xe (P 0.135, R 0.083, mAP50 0.103).

**Quan sát về quá trình train:** `best.pt` được chọn ở **epoch 1** (mAP50–95 = 0.2018), các epoch sau không vượt qua; epoch 50 chỉ đạt mAP50–95 = 0.1262 (xem [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv)). Log ghi 3/4 hàng của đầu phân lớp được lấy từ trọng số tiền huấn luyện theo tên lớp, nên số liệu cuối chủ yếu phản ánh trọng số tiền huấn luyện, không phải thứ model học thêm được từ 50 ảnh. Tập eval chỉ có 113 box nên số liệu dao động mạnh giữa các epoch.

Ảnh và chỉ số: [ảnh dự đoán trên 10 ảnh eval](assets/eval_predictions/visuals/), [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png).

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?** `bus` có recall 0, nhưng eval chỉ có 2 xe `bus` (ở `drive_051` và `drive_058`) nên khó kết luận. Đáng chú ý hơn là `van`, recall 0.083 trên 12 xe. Ảnh minh họa: [drive_058](assets/eval_predictions/visuals/drive_058.jpg), nhãn gold có 32 `car`, 6 `van`, 1 `bus`, nhưng model dự đoán 20 `car`, 3 `bus` và **0 `van`**. Ở [drive_051](assets/eval_predictions/visuals/drive_051.jpg) gold có 1 `bus`, model dự đoán 8 `car` và 1 `van`, không có `bus`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?** Cần kiểm độ khít của box ở ngưỡng IoU cao: box lỏng hoặc chặt không nhất quán, box ở xe bị che hoặc bị cắt rìa, và nhất quán giữa các ảnh. Ở bài này mAP50 chưa cao (0.272), mAP50–95 bằng khoảng 74% mAP50 (lớp `car`: 0.740 so với 0.523), tức vấn đề chính là chưa phát hiện đủ xe hơn là box lệch. Mean IoU nhãn của tôi so với gold là 0.7745, đạt 0.70 nhưng chưa đạt mức xuất sắc 0.80, nên độ sát của box vẫn có thể cải thiện.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ?** Trong confusion matrix, **11/12 xe `van` bị đoán thành `bus`** và 4/5 xe `truck` bị đoán thành `car`. Điều này khớp với việc ranh giới `van`/`bus` và `truck`/`car` là chỗ tôi ghi mơ hồ trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Chưa chắc nguyên nhân nằm ở guideline: `van` và `truck` chỉ có 59 và 20 box trong nhãn train, nên mất cân bằng lớp và số mẫu ít cũng có thể giải thích. Cột "background" trong confusion matrix có số rất lớn vì ma trận tính ở ngưỡng tin cậy thấp, tôi không dùng cột này để kết luận.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation?** Eval là ảnh model chưa từng học, để đo khả năng tổng quát hóa; đưa vào train thì số đo không còn ý nghĩa. Trong notebook, 10 ảnh này cũng làm validation mỗi epoch và để chọn `best.pt`, nên chúng không phải test set độc lập.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CVAT local, sau khi bật lại Docker | "Cannot connect to the server" | Kiểm tra container, chờ vài phút cho server khởi động xong rồi tải lại trang | Hết lỗi, CVAT truy cập bình thường |
| Chấm nhãn với gold | F1 0.4808, precision 0.359 do dư nhiều box xe nhỏ | Dọn box xe nhỏ ở xa trong CVAT, export lại, chấm lại | F1 0.5453, vẫn dưới 0.80 trong tài liệu; FP còn 606 . nhưng phù hợp theo yêu cầu 0.5 mới của labcoach|
| Colab | Model train 50 epoch không cải thiện so với epoch 1 | Giữ nguyên tham số notebook theo yêu cầu | Ghi nhận ở mục 5; chưa thử thay đổi dữ liệu hoặc tham số |

## 7. Tự đánh giá theo rubric

- **Hoàn thành và định dạng YOLO (10):** có, 50 file, checker đạt 50/50.
- **Đầy đủ vật thể (20) và bám sát box (20):** mean IoU 0.7745 cho thấy box khớp khá tốt; nhưng còn 181 FN và 606 FP nên độ đầy đủ chưa đạt. Chưa kiểm hết từng ảnh.
- **Phân lớp nhất quán (15):** chưa chắc; model nhầm `van` thành `bus`, và tôi chưa xác nhận được nhãn train đã nhất quán ở các ca van/bus.
- **Mini guideline (10):** có định nghĩa, quy tắc và bốn ca mơ hồ. Một số quyết định (xe cứu hộ → `truck`) cần giảng viên xác nhận.
- **Gold QC (15):** có chạy và có rework. F1 0.5453, mean IoU 0.7745. F1 vượt ngưỡng 0.5 của lab coach nhưng chưa đạt 0.80 trong tài liệu.
- **Pipeline YOLO (10):** Colab chạy xong, không dùng eval để train, có metrics và ảnh eval; phân tích ở mục 5. Chưa thử cải thiện model.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ:** Không có
- **Dùng ở bước nào:** Không có
- **Phần đã tự kiểm tra:** gán nhãn 50 ảnh, đọc và hoàn thành theo repo
- **Nguồn ngoài repo:** không có.
