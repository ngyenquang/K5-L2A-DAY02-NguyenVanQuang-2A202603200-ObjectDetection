# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

Ảnh train: CCTV giao thông, kích thước 640×640. Schema cố định: `car=0, truck=1, bus=2, van=3`.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải cabin nhỏ. Thân xe thấp, một khối, không có khoang thùng sau cabin lớn. | Taxi màu vàng/xanh, xe 4–5 chỗ phổ thông. Không gán xe khách hay xe tải. |
| `truck` | Xe tải chở hàng và xe chuyên dụng cabin lớn kiểu xe tải (xe thùng, xe cứu hộ có cần cẩu, xe đầu kéo). Thân dài hơn hẳn xe con, có thùng hàng hoặc thiết bị phía sau cabin. | Xe thùng chở hàng; xe cứu hộ của công an trong `drive_019`. |
| `bus` | Xe buýt và minibus chở khách cỡ lớn: thân rất dài, nhiều cửa sổ dọc thân, cao hơn xe con. | Xe buýt thành phố hai màu, xe buýt hai khúc. |
| `van` | Xe van/minivan chở hàng hoặc chở người cỡ vừa: thân hộp, cao hơn xe con nhưng ngắn và thấp hơn xe buýt, thường có cửa trượt và phần đuôi vuông. | Minivan bạc/trắng trong `drive_024`, `drive_019`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: cạnh box chạm sát phần thân xe nhìn thấy được, không lấy thêm mặt đường, bóng đổ hay xe bên cạnh. Mỗi xe một box riêng, kể cả cùng lớp.
- Vật thể bị che khuất: vẫn vẽ box quanh phần nhìn thấy được nếu nhận ra được lớp. Hai xe chồng nhau thì vẽ hai box riêng.
- Vật thể bị cắt bởi rìa ảnh: box chạm đúng rìa ảnh, không đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: không gán nếu xe quá nhỏ ở xa hoặc chỉ lộ một mảnh, không chắc chắn lớp. Không gán người đi bộ, xe đạp, xe máy/mô tô, xe ba bánh, biển báo và vật ngoài bốn lớp.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_019` | Xe cứu hộ của công an: thân trắng, cabin lớn, có cần cẩu phía sau. | Gán `truck`. | Là xe chuyên dụng cabin lớn, thân dài, có thiết bị phía sau, gần xe tải hơn xe con hay bán tải cabin nhỏ. Cần labcoach xác nhận vì schema không nêu rõ xe chuyên dụng. |
| `drive_024` | Xe ba bánh chở người ở giữa đường, nhìn giống xe có mui. | Không gán. | Không thuộc bốn lớp. Cùng nhóm với mô tô/xe máy mà GUIDE.md quy định không gán. |
| `drive_028`, `drive_044`, `drive_014` | Rất nhiều xe nhỏ ở xa, chen sát nhau, chỉ vài chục pixel. Bản đầu tôi vẽ hết (ví dụ drive_028: 24 box, drive_044: 42 box). | Khi rework, bỏ các xe quá nhỏ ở xa (drive_028 còn 14 box, drive_044 còn 24 box). | Chúng khó xác định lớp chắc chắn, đúng quy tắc "không tự đoán". Trong toàn bộ 250 dòng nhãn bị bỏ, cạnh dài trung vị khoảng 19 px, 75% không quá 28 px; box giữ lại có cạnh dài trung vị khoảng 49 px. Đây là quy tắc ước lượng, không phải ngưỡng cứng. |
