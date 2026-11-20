## 1. Bối cảnh và Bài toán thực tế

*(Lưu ý nhỏ: Trong đề bài có nhắc chữ "cửa hàng", em hiểu đây là mô hình trung tâm đào tạo / cửa hàng bán khóa học kỹ năng cần tự động hóa khâu kèm cặp học viên).*

Trong quá trình học tập và tự nghiên cứu, nhiều bạn học viên thường học vẹt lý thuyết mà không thực sự hiểu bản chất. Nếu áp dụng **kỹ thuật Feynman** (bắt học viên tự giải thích lại kiến thức bằng ngôn ngữ đơn giản nhất như đang giảng cho người chưa biết gì) thì giảng viên lại không đủ thời gian ngồi đọc và chấm từng bài giải thích của hàng trăm bạn.

Vì vậy, em thiết kế một giải pháp dùng **Feynman Prompt gia sư** kết hợp với **Rubric chấm bài tự động**:

* AI đóng vai một người gia sư kiên nhẫn, nhận bài giải thích của học viên, đối chiếu với tài liệu chuẩn và chấm điểm theo thang Rubric (Tính chính xác, Tính đơn giản/dễ hiểu, Ví dụ minh họa).

* Đồng thời, Prompt phải có câu lệnh điều kiện để chặn các bẫy dữ liệu như: học viên nộp bài trống, copy-paste nguyên văn sách giáo khoa (không tự diễn đạt), hoặc giải thích lạc đề.

## 2. Bước 1: Thiết kế giải pháp (Input / Output)

### Đầu vào (Input)

Gồm 3 thông tin bắt buộc đưa vào hệ thống:

1. `{{CHU_DE_KIEN_THUC}}`: Tên khái niệm hoặc định nghĩa chuẩn mà học viên cần giải thích.

2. `{{BAI_GIAI_THICH_HOC_VIEN}}`: Đoạn văn do học viên tự viết lại bằng lời của mình theo phương pháp Feynman.

3. `{{THANG_DIEM_RUBRIC}}`: Các tiêu chí chấm điểm (Tổng $10$ điểm: Hiểu đúng bản chất $4$đ, Diễn đạt đơn giản $3$đ, Có ví dụ thực tế $3$đ).

### Đầu ra mong muốn (Output)

Kết quả AI trả về được chia làm 2 phần rõ ràng:

* **Phần 1: Kiểm tra tính hợp lệ (Bắt lỗi dữ liệu):**

  * Nếu bài làm rỗng, quá ngắn (dưới 15 từ), sai chủ đề, hoặc copy y hệt định nghĩa mẫu $\rightarrow$ Xuất cảnh báo lỗi cụ thể và yêu cầu học viên làm lại (không chấm điểm).

* **Phần 2: Bảng chấm điểm Rubric & Gợi ý Feynman (Nếu bài hợp lệ):**

  * Bảng chấm điểm chi tiết từng tiêu chí kèm nhận xét lỗi sai.

  * Lời khuyên của gia sư AI chỉ ra "lỗ hổng kiến thức" (chỗ nào học viên còn giải thích mơ hồ) để học viên tự sửa lại cho hoàn thiện.

## 3. Bước 2: Viết Prompt và Chạy thử

### Thiết kế Prompt hoàn chỉnh

```
Bạn là một Gia sư AI thân thiện, áp dụng phương pháp học tập Feynman để kiểm tra mức độ hiểu bài của học viên và chấm điểm theo Rubric.

### DỮ LIỆU ĐẦU VÀO:
- Chủ đề & Kiến thức chuẩn: "{{CHU_DE_KIEN_THUC}}"
- Bài giải thích của học viên: "{{BAI_GIAI_THICH_HOC_VIEN}}"

### BƯỚC 1: KIỂM TRA BẪY DỮ LIỆU (RÀNG BUỘC ĐIỀU KIỆN)
Trước khi chấm điểm, hãy kiểm tra bài làm của học viên theo các điều kiện sau:
- Điều kiện 1 (Dữ liệu trống/quá ngắn): Nếu bài làm để trống, chỉ có ký tự rác, hoặc ngắn dưới 15 từ -> Dừng chấm, xuất thông báo: "[LOI_DU_LIEU]: Bài làm quá ngắn hoặc bị trống, bạn vui lòng tự diễn đạt lại kiến thức bằng lời của mình nhé!"
- Điều kiện 2 (Bẫy copy-paste / học vẹt): Nếu bài làm giống trên 80% câu chữ của phần "Kiến thức chuẩn" mà không có sự diễn đạt lại bằng ngôn ngữ đời thường -> Dừng chấm, xuất thông báo: "[CANH_BAO_FEYNMAN]: Bạn đang chép lại định nghĩa mẫu. Hãy thử giải thích như đang nói chuyện với một học sinh cấp 2 nhé!"
- Điều kiện 3 (Lạc đề): Nếu nội dung không liên quan gì đến chủ đề -> Dừng chấm và nhắc học viên nộp đúng bài.

### BƯỚC 2: CHẤM ĐIỂM THEO RUBRIC & PHẢN HỒI GIA SƯ (Chỉ chạy khi vượt qua Bước 1)
Hãy đánh giá bài làm dựa trên Rubric 10 điểm dưới đây:
1. Tính chính xác của bản chất kiến thức (Tối đa 4 điểm): Không sai lệch khái niệm cốt lõi.
2. Ngôn ngữ đơn giản, dễ hiểu (Tối đa 3 điểm): Tránh từ ngữ hàn lâm khó hiểu, giải thích mượt mà.
3. Ví dụ minh họa thực tế (Tối đa 3 điểm): Có ví dụ gần gũi giúp người nghe hình dung ngay.

### ĐỊNH DẠNG ĐẦU RA BẮT BUỘC:
- Trạng thái kiểm tra: (Hợp lệ / Lỗi)
- Bảng điểm Rubric (Markdown Table gồm: Tiêu chí | Điểm đạt được | Nhận xét chi tiết)
- Tổng điểm: .../10
- Góc gia sư Feynman: Chỉ ra 1 lỗ hổng kiến thức cần bổ sung và đặt 1 câu hỏi gợi mở để học viên tự suy nghĩ thêm.

```

### Kết quả chạy thử thực tế (2 trường hợp)

#### Thử nghiệm 1: Kiểm tra bẫy dữ liệu (Học viên copy-paste nguyên văn định nghĩa)

* **Input đưa vào:**

  * *Chủ đề & Kiến thức chuẩn:* Khái niệm API (Application Programming Interface) là giao diện lập trình ứng dụng, cho phép các phần mềm giao tiếp và trao đổi dữ liệu với nhau thông qua các quy tắc và giao thức xác định.

  * *Bài giải thích của học viên:* `"API là giao diện lập trình ứng dụng, cho phép các phần mềm giao tiếp và trao đổi dữ liệu với nhau thông qua các quy tắc và giao thức xác định."`

* **Kết quả AI phản hồi:**

  > **Trạng thái kiểm tra:** Lỗi
  >
  > `[CANH_BAO_FEYNMAN]`: Bạn đang chép lại định nghĩa mẫu. Theo phương pháp Feynman, việc lặp lại nguyên văn câu chữ hàn lâm chứng tỏ bạn chưa thực sự biến kiến thức thành của mình. Hãy thử giải thích lại khái niệm API như đang kể cho một người bạn không học chuyên ngành IT nghe nhé (ví dụ: so sánh với người phục vụ bàn ở nhà hàng)!

#### Thử nghiệm 2: Bài làm hợp lệ của học viên (Chấm điểm Rubric & Gợi ý)

* **Input đưa vào:**

  * *Chủ đề & Kiến thức chuẩn:* Khái niệm API (Giao diện lập trình ứng dụng cho phép các phần mềm giao tiếp và trao đổi dữ liệu với nhau).

  * *Bài giải thích của học viên:* `"Hiểu đơn giản thì API giống như anh nhân viên phục vụ ở quán ăn vậy. Bạn (khách hàng) ngồi ở bàn muốn gọi món thì không tự chạy vào bếp lục nồi niêu được, mà phải đưa yêu cầu cho anh phục vụ. Anh phục vụ sẽ mang yêu cầu đó vào nhà bếp (hệ thống dữ liệu) để đầu bếp nấu, sau đó bưng món ăn ra lại cho bạn. Nhờ có API mà hai ứng dụng khác nhau có thể nói chuyện và lấy thông tin của nhau mà không cần biết bên trong code của nhau viết gì."`

* **Kết quả AI phản hồi:**

  > **Trạng thái kiểm tra:** Hợp lệ
  >
  > **Bảng điểm Rubric:**
  >
  > | Tiêu chí | Điểm đạt được | Nhận xét chi tiết | 
  > | ----- | ----- | ----- | 
  > | **1. Tính chính xác của bản chất kiến thức** | $3.5 / 4.0$ | Nắm đúng bản chất API là cầu nối trung gian nhận yêu cầu (request) và trả về kết quả (response) giữa các hệ thống mà không lộ mã nguồn bên trong. | 
  > | **2. Ngôn ngữ đơn giản, dễ hiểu** | $3.0 / 3.0$ | Cách dùng từ đời thường rất tự nhiên, người không biết gì về lập trình đọc vào cũng hiểu ngay. | 
  > | **3. Ví dụ minh họa thực tế** | $3.0 / 3.0$ | Ví dụ về "anh nhân viên phục vụ quán ăn" cực kỳ kinh điển và trực quan, khớp hoàn toàn với cơ chế hoạt động của API. | 
  >
  > **Tổng điểm:** $9.5 / 10$
  >
  > **Góc gia sư Feynman:**
  > * **Lỗ hổng nhỏ cần bổ sung:** Bạn đã giải thích rất hay quy trình đưa yêu cầu và nhận kết quả, nhưng chưa nhắc tới yếu tố "quy tắc/bảo mật" (ví dụ: anh phục vụ chỉ nhận gọi những món có trong thực đơn chứ không nhận yêu cầu vô lý).
  > * **Câu hỏi gợi mở:** Theo bạn, nếu một ứng dụng lạ gửi yêu cầu lấy dữ liệu mật khẩu người dùng qua API thì điều gì sẽ xảy ra, và "anh phục vụ API" cần kiểm tra thứ gì trước khi cho phép vào bếp?
