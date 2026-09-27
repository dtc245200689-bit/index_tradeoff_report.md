# BÁO CÁO ĐÁNH ĐỔI HIỆU NĂNG CHỈ MỤC (INDEX TRADEOFF REPORT)

## 1. Lập luận về sự dư thừa của Covering Index trong hệ thống ghi cao (High-Write)
Trong hệ thống SmartFactory với lưu lượng dữ liệu IoT đổ về liên tục, việc lạm dụng Covering Index (đưa quá nhiều cột như `temperature`, `humidity` vào chỉ mục) tạo ra gánh nặng tài nguyên rất lớn. Dù giúp câu lệnh `SELECT` đạt tốc độ tối đa nhờ trạng thái `Using index` (không cần quay lại đĩa đọc bảng thô), nó lại làm kích thước `Index Length` phình to bất thường. 

## 2. Tính toán dung lượng lưu trữ (Byte Calculation) minh họa
Giả sử hệ thống nhận **10,000,000 dòng dữ liệu** mỗi ngày:
- **Cột `status` dùng VARCHAR(20) utf8mb4:** Tiêu tốn tối đa \(20 \times 4 + 1 = 81 \text{ bytes}\) cho mỗi bản ghi trong nút Index.
- **Nếu tối ưu hóa thay thế `status` sang TINYINT:** Kiểu dữ liệu này chỉ chiếm đúng **1 byte**. 

**Số liệu so sánh cục bộ tầng vật lý đĩa cứng:**
- **Giải pháp cũ (Covering Index):** Kích thước mỗi bản ghi khóa trong Secondary Index lên đến khoảng \(120 \text{ bytes}\) (bao gồm cả các cột data kéo theo). Tổng dung lượng Index tăng thêm mỗi ngày: \(\approx 1.2 \text{ GB}\).
- **Giải pháp tinh gọn (2 cột lọc + Chuyển sang TINYINT):** Kích thước bản ghi khóa giảm xuống chỉ còn khoảng \(15 \text{ bytes}\) (DATETIME 5 bytes + TINYINT 1 byte + PK Pointer 8 bytes). Tổng dung lượng Index mỗi ngày giảm xuống: \(\approx 150 \text{ MB}\).

**Kết luận:** Chấp nhận giảm một chút tốc độ đọc (phải tốn thêm chi phí Look up table quay lại đĩa đọc dữ liệu thô) để giải phóng hệ thống khỏi thảm họa phình to bộ nhớ, cứu vãn băng thông I/O đĩa cứng và tăng tốc độ xử lý câu lệnh `INSERT` lên gấp nhiều lần.
