# Giới thiệu về workflow trong cinder

# 1 Thành phần Cinder <a name="1"> </a>

- Chúng ta có 4 quy trình tạo nên Cinder Service :

|Process|mô tả|
|---------|-----|
|Cinder-api|là một ứng dụng WSGI chấp nhận và xác nhận các yêu cầu REST (JSON hoặc XML) từ client và chuyển chúng tới các quy trình CInder khác nếu thích hợp với AMQP|
|Cinder-scheduler|Chương trình lập lịch các định back-end nào sẽ là điểm đến cho một yêu cầu tạo ra volume hoặc chuyển yêu cầu đó. Nó duy trì trạng thái không liên tục cho các back-end (Ví dụ : khả năng sẵn có , khả năng và các thông số kỹ thuật được hỗ trợ) có thể được tận dụng khi đưa ra các quyết định về vị trí . Thuật toán được sử dụng bởi chương trình lên lịch có thể được thay đổi thông qua cấu hình Cinder|
|Cinder-volume|Cinder-volume chấp nhận các yêu cầu từ các quy trình CInder khác đóng vai trò là thùng chứa hoạt động cho các trình điều khiển Cinder. Quá trình này là đa luồng và thường có một luồng thực hiện trên mỗi Cinder back-end giống như định nghĩa trong tập tin cấu hình Cinder.|
|Cinder-backup|Xử lý tương tác với các mục tiêu sao có khả năng sao lưu (Ví dụ như OpenStack Object Storage Service - Swift). Khi một máy client yêu cầu sao lưu volume được tạo ra hoặc quản lý.|

# 2 Workflow của Cinder khi tạo mới volume <a name="2"> </a>

<img src="https://i.imgur.com/foE7pok.png">

1. Client yêu cầu tạo volume thông qua REST API của cinder ( client sử dụng python-cinderclient hoặc thông qua dashboard)
2. cinder-api xác thực yêu cầu hợp lệ ko?, thông tin user. Mội khi được xác thực, put messagelên AMQP queue để xử lý.
3. cinder-volume xử lý message của queue, gửi message cho cinder-scheduler để xác định backend cung cấp cho volume.
4. cinder-scheduler xử lý message của queue, sinh ra danh sách các ứng viên (node storage) dựa trên trạng thái hiện tại và yêu cầu tiêu chí về volume criteria (size, availability zone, volume type (including extra specs)).
5. cinder-volume đọc thông tin trả lời cinder-scheduler từ queue, lặp lại danh sách ứng viên bằng phương thức backend driver cho đến khi thành công.
6. Cinder driver tạo yêu volume được yêu cầu thông qua tương tác với hệ thống con storage ( phụ thuộc vào cấu hình và giao thức)
7. cinder-volume tập hợp volume metadata từ queue và kết nối thông tin và chuyển message trả lời đến AMQP queue.
8. cinder-api đọc message trả lời từ queue và trả lời client.
9. Client nhận được thông tin bao gồm trạng thái của yêu cầu tạo volume: volume UUID, etc.
