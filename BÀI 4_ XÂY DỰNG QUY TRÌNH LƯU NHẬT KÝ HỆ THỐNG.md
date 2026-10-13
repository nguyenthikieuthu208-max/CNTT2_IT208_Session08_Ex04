***BÀI 4: XÂY DỰNG QUY TRÌNH LƯU NHẬT KÝ HỆ THỐNG***

Phần 1: Bảng so sánh toán tử \> và \>\> trong PowerShell

Để hiểu rõ sự đánh đổi (Trade-off) giữa hai toán tử điều hướng dữ liệu (Redirect Output), dưới đây là bảng so sánh chi tiết dựa trên cơ chế hoạt động của PowerShell:

| Tiêu chí so sánh | Toán tử \> (Ghi đè \- Override) | Toán tử \>\> (Ghi nối tiếp \- Append) |
| ----- | ----- | ----- |
| Bản chất lệnh gốc | Là Alias của lệnh Out-File | Là Alias của lệnh Out-File \-Append |
| Cơ chế hoạt động | Tự động xóa sạch toàn bộ nội dung cũ của tệp tin hiện tại (nếu tệp đã tồn tại), sau đó mới ghi đè dữ liệu mới vào từ đầu. | Giữ nguyên vẹn nội dung cũ, nhảy xuống dòng cuối cùng của tệp tin và ghi nối tiếp dữ liệu mới vào đuôi. |
| Rủi ro mất dữ liệu | Cực kỳ cao. Chỉ cần gõ nhầm một lần, toàn bộ dữ liệu lịch sử tích lũy trước đó sẽ bị xóa trắng hoàn toàn. | Cực kỳ thấp. An toàn cho dữ liệu lịch sử vì tệp tin liên tục được bồi đắp thêm theo thời gian. |
| Ngữ cảnh sử dụng | Phù hợp khi làm Báo cáo ca sáng / Báo cáo ngày lẻ. Mỗi ngày chỉ cần một file mới tinh, độc lập, không quan tâm dữ liệu hôm qua. | Phù hợp cho việc Theo dõi hệ thống 24/7. Ghi nhận log lỗi phát sinh liên tục trong nhiều ngày, nhiều tháng để phục vụ tra cứu. |
| Định dạng Encoding mặc định | Mặc định ghi tệp theo chuẩn UTF-16LE (gây lỗi phông chữ nếu mở bằng các công cụ Linux/CMD cũ). | Mặc định ghi tệp theo chuẩn UTF-16LE (gây lỗi phông chữ nếu mở bằng các công cụ Linux/CMD cũ). |

Phân tích bẫy dữ liệu (Cuối tháng gõ nhầm \> thay vì \>\>):  
Nếu An gõ nhầm toán tử \> vào ngày cuối tháng, toàn bộ dữ liệu nhật ký tích lũy của 30 ngày trước đó sẽ bị xóa sạch sành sanh vĩnh viễn trong vòng 1 giây. File daily\_report.txt lúc này sẽ chỉ còn trơ trọi dữ liệu lỗi của ngày cuối cùng, gây ra thảm họa mất dữ liệu báo cáo nghiêm trọng cho phòng DevOps.

Phần 2: Lựa chọn giải pháp tối ưu cho hệ thống ghi log

1\. Chốt giải pháp tối ưu:

Đối với việc ghi nhật ký lỗi phát sinh liên tục trong ngày và duy trì hệ thống bền vững, giải pháp tối ưu nhất là sử dụng lệnh Add-Content kèm tham số \-Encoding utf8.

* Câu lệnh chuẩn khuyến nghị:

powershell  
Get-Content server.log | Select-String "ERROR" | Add-Content \-Path daily\_report.txt \-Encoding utf8

2\. Lập luận lý do lựa chọn (Vì sao không dùng \>\>):

Loại bỏ rủi ro gõ nhầm ký tự: Khi viết mã kịch bản tự động, việc viết tường minh bằng chữ câu lệnh Add-Content hoặc tham số \-Append sẽ giúp lập trình viên không bao giờ bị dính bẫy gõ nhầm một ký tự \> tai hại như khi dùng toán tử viết tắt.

Chuẩn hóa định dạng Encoding quốc tế: Toán tử \>\> mặc định xuất file dạng UTF-16, định dạng này rất kén các công cụ đọc log tự động của hệ thống Linux hoặc các phần mềm quản lý log tập trung (như ELK Stack, Splunk). Việc ép cấu hình về \-Encoding utf8 giúp tệp tin hiển thị chuẩn xác 100% phông chữ, dễ dàng đọc, phân tích dữ liệu xuyên suốt trên mọi hệ điều hành (Windows, Linux, macOS) mà không bị lỗi ký tự rác.

