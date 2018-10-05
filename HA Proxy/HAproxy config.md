# cấu hình file HAproxy

## 1. Cách thức hoạt động HAproxy

HAProxy là single-threaded, event-driven, non-blocking engine kết hợp các I/O layer với priority-based scheduler. Vì nó được thiết kế với mục tiêu vận chuyển dữ liệu, kiến trúc của nó được tối ưu hóa để chuyển dữ liệu nhanh nhất có thể. Nó có những layer model với những cơ chế riêng để đảm bảo dữ liệu không đi tới những level cao hơn nếu không cần thiết. Phần lớn những quá trình xử lí diễn ra ở kernel và HAProxy làm mọi thứ tốt nhất để giúp kernel làm việc nhanh nhất có thể.

HAProxy (product) chỉ yêu cầu haproxy (programs) được thực thi và file cấu hình để chạy. File cấu hình sẽ được đọc trước khi nó khởi động, sau đó HAProxy sẽ cố gắng gộp tất cả listening sockets và sẽ từ chối khởi động nếu có lỗi.

Một khi HAProxy được bật lên, nó thực hiện 3 điều:

- Xử lí các incoming connections
- Định kì check các trạng thái của server (health checks)
- Trao đổi thông tin với các haproxy nodes khác

Quá trình xử lí các incoming connections là phần phức tạp nhất và nó phụ thuộc vào rất nhiều cấu hình. Có thể tóm gọn nó lại trong 9 bước:

- Cho phép các connections từ listening sockets thuộc về các cấu hình trong phần frontend
- Áp dụng các frontend-specific processing rules đối với những connections này
- Chuyển các incoming connections tới "backend", nơi chứa các server và cả những kế hoạch load balancing cho nó
- Áp dụng những backend-specific processing rules cho các connections
- Dựa vào kế hoạch load balancing mà quyết định server sẽ nhận kết nối
- Áp dụng backend-specific processing rules cho dữ liệu được trả về từ server
- Áp dụng frontend-specific processing rules cho dữ liệu được trả về từ server
- Tạo log để ghi lại những gì đã xảy ra
- Đối với http, lặp lại bước 2 để đợi một request mới, nếu không có, tiến hành đóng kết nối.

