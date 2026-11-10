# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô cá nhân chở người từ 4 đến 7 chỗ ngồi, thân vỏ liền khối (unibody), không có thùng chở hàng rời. | Xe sedan, hatchback, coupe, và các dòng xe SUV/Crossover gầm cao phục vụ gia đình. |
| `truck` | Xe cơ giới chuyên dụng dùng để vận chuyển hàng hóa, tải trọng từ nhẹ đến nặng. Thường có cabin lái tách rời thùng hoặc rơ-moóc hàng. | Xe tải thùng, xe ben, xe bồn, xe đầu kéo container. Xe bán tải (pickup như Ford Ranger, Hilux) xếp vào lớp `truck`. |
| `bus` | Xe buýt công cộng nội đô, xe khách liên tỉnh, xe du lịch đường dài cỡ lớn có sức chứa từ 24 chỗ ngồi trở lên. | Thân xe dạng hộp chữ nhật cỡ lớn, trần xe rất cao, nhiều hàng cửa sổ kính dài, trục cơ sở dài vượt trội. |
| `van` | Xe thương mại cỡ trung 9–16 chỗ hoặc xe tải van nhỏ có khoang lái và khoang chở hàng/chở khách liền một khối kín. | Trần cao hơn xe con (`car`) nhưng thân xe ngắn và thấp hơn xe khách lớn (`bus`). Ví dụ: Ford Transit, Hyundai Solati, Toyota HiAce, xe van giao hàng. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:**
  - 4 cạnh của Bounding Box phải tiếp xúc chính xác pixel biên ngoài cùng của thân xe (bao gồm: gương chiếu hậu, cản trước/sau, giá nóc, bánh xe tiếp đất).
  - Tuyệt đối **KHÔNG** bao gồm bóng đổ của xe dưới mặt đường (shadow), khói xả từ ống pô, hoặc vệt sáng đèn pha.
- **Vật thể bị che khuất:**
  - Chỉ vẽ Bounding Box bám sát **phần thân xe thực tế nhìn thấy được** nếu phần nhìn thấy còn $\ge 20\%$ và đủ đặc điểm nhận diện rõ lớp.
  - Không vẽ ước lượng lấn sang xe che khuất bên cạnh. Nếu bị che khuất $> 80\%$ không thể nhận dạng chắc chắn thì bỏ qua.
- **Vật thể bị cắt bởi rìa ảnh:**
  - Kéo cạnh Bounding Box áp sát đúng rìa ảnh khung hình (`x = 0` hoặc `x = 1`, `y = 0` hoặc `y = 1`). Không suy đoán hay vẽ vượt ra ngoài biên ảnh.
  - Bỏ qua nếu phần vật thể lọt vào trong ảnh quá nhỏ ($< 15\%$ diện tích xe).
- **Vật thể rất nhỏ/mơ hồ:**
  - Bỏ qua các xe ở quá xa có kích thước nhỏ hơn $15 \times 15\text{ px}$.
  - Bỏ qua vật thể bị nhòe chuyển động (motion blur) nặng hoặc ngược sáng tối đen mà mắt người không thể tự tin phân biệt được là loại xe nào.
  - Tuyệt đối không gán xe máy, xe đạp, người đi bộ, biển báo hoặc hình ảnh xe trên biển quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định bạn đã làm | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_006` | Chiếc xe màu bạc có giá nóc ở góc dưới bên phải làn đường (#88), phân vân giữa xe 7 chỗ gầm cao (`car`) và xe `van`. | **van** | Thân xe có kết cấu khối hộp vuông vức (boxy), trần xe cao và có giá nóc chở hàng đặc trưng của dòng xe minivan/van nhỏ. Quyết định phân loại vào lớp `van` (label 3). |
| `drive_006` | Chiếc xe hatchback màu trắng ở góc dưới bên trái (#86), di chuyển giữa ngã tư ướt mưa. | **car 86** | Xe con 4 chỗ gầm thấp, thân vỏ unibody liền khối phục vụ cá nhân/gia đình. Đúng định nghĩa chuẩn của lớp `car` (label 0). |
| `drive_001` | Hàng loạt xe nhỏ li ti đỗ ở hậu cảnh tít xa cuối đường, kích thước $< 15 \times 15\text{ px}$, nhòe mờ không rõ chi tiết đèn/biển số (ban đầu gán tới 16 xe). | **Bỏ qua (không gán nhãn)** | Áp dụng quy tắc "Vật thể rất nhỏ/mơ hồ": Bỏ qua các xe ở quá xa $< 15 \times 15\text{ px}$. Đã xóa các box thừa này để giảm False Positive (FP), giúp F1 đạt chuẩn. |
