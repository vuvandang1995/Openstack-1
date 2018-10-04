# Giới thiệu về 1 số câu lệnh thông dụng và bảng của netstat

## 1. Netstat -an (Hiển thị tất cả kết nối và lắng nghe, hoặc Netstat -n để chỉ lấy các thông tin đang kết nối)

<img src="https://i.imgur.com/UzBiHyw.png">

- Proto : Giao thức được sử dụng truy cập

- Local Address : Gồm có địa chỉ IP:port của máy tính hiện tại đang sử dụng (ip local), nếu cổng chưa được thành lập sẽ thể hiện là dấu (*)

- Foreign : Địa chỉ IP:Port của Destination mà máy tính đang truy cập, nếu cổng chưa được thành lập sẽ thể hiện là dấu (*)

- State : Trạng thái kết nối

|State                              | Description
|-----------------------------------|------------------
|CLOSED| Chỉ ra rằng máy chủ đã nhận được tín hiệu ACK từ máy khách và kết nối bị đóng
|CLOSED_WAIT| Chỉ ra rằng máy chủ đã nhận được tín hiệu FIN đầu tiên từ máy khách và kêt nối đang đóng
|ESTABLIED|Chỉ ra rằng máy chủ đã nhận được tín hiệu SYN từ máy khách và phiên kết nối được thiết lập và trao đổi dữ liệu
|TIME_WAIT| Chỉ ra rằng máy khách nhận ra kết nối đang hoạt động nhưng không được sử dụng, và kết nối đang chờ giải phóng
|LISTENING|Chỉ ra rằng máy chủ đang sẵn sàng chờ kết nối
|FIN_WAIT_1|Chỉ ra rằng kết nối vẫn hoạt động nhưng không được sử dụng
|FIN_WAIT_2|Chỉ ra rằng máy khách vừa nhận được tín hiệu FIN đầu tiên từ máy chủ
|LAST_ACK|Chỉ ra rằng máy chủ đang trong quá trình gửi tín hiệu FIN của nó
|SYN_RECEIVED|Chỉ ra rằng máy chủ vừa nhận được tín hiệu SYN từ máy khách
|SYN_SEND|Chỉ ra rằng kết nối này đang mở và đang hoạt động

## 2. Netstat -e (Hiển thị thống kê card mạng ethernet)

<img src="https://i.imgur.com/xW3LH7i.png">

## 3. Netstat -r (Hiển thị thông tin bảng định tuyến)

<img src="https://i.imgur.com/RUuWzGV.png">

<img src="https://i.imgur.com/BJiXb78.png">

- Interface list: Hiển thị thông tin tên card mạng và địa chỉ MAC tương ứng của card mạng đó
- Active route: 
<ul>
  <ul>
    <li>Network destination:Địa chỉ đích</li>
    <li>Netmask: subnetmask</li>
    <li>gateway: gateway của dải mạng, on-link có nghĩa là đây nhưng địa chỉ cục bộ có thể kết nối trực tiếp với nhau nên không cần gateway </li>
    <li>Interface: Địa chỉ nguồn của thiết bị</li>
    <li>Metric: Độ ưu tiên của đường đi</li>
  </ul>
  </ul>

## 4. Netstat -f (Hiển thị các kết nối đang hoạt động và hiển thị domain thay cho địa chỉ ip đích, với netstat chỉ hiển thị tên miền đại diện)

<img src="https://i.imgur.com/KoGObwK.png">

## 5. Netstat -on(Hiển thị thông tin Process ID của tiến trình)

<img src="https://i.imgur.com/V3irPCr.png">

## 6. Netstat -s (Hiển thị thống kê cho các giao thức )

<img src="https://i.imgur.com/JAogMMJ.png">

|units|description
|-----|-------------------------------------------
|Packets Received| Tổng số gói đầu vào nhận được từ các interfaces
|Received Header Errors|Tổng số lượng các gói dữ liệu đầu vào bị loại bỏ do lỗi trong các tiêu đề IP của chúng.
|Received Address Errors|Số lượng các gói dữ liệu đầu vào bị loại bỏ vì địa chỉ IP trong trường đích của tiêu đề IP của chúng không hợp lệ
|Datagrams Forwarded|Số lượng các gói dữ liệu đầu vào đã được chuyển tiếp đến đích 
|Unknown Protocols Received|Số lượng gói dữ liệu bị loại bỏ do giao thức không xác định hoặc không được hỗ trợ
|Received Packets Discard|Số lượng các gói dữ liệu đầu vào đã bị loại bỏ mà không được tính trong một bộ đếm đầu vào khác
|Received Packets Delivered|Tổng số lượng dữ liệu đầu vào được chuyển giao thành công tới giao thức người dùng IP
|Output Request|Tổng số IP mà các giao thức người dùng IP cục bộ được cung cấp cho IP trong các yêu cầu truyền
|Reassembly Required|Số lượng các đoạn IP nhận được cần thiết để được tập hợp lại
|Reassembly Successful|Số lượng các gói dữ liệu IP được tái hợp thành công
|Reassembly Failures|Số lỗi được phát hiện bởi thuật toán reassembly IP
|Datagrams Sucessfully Fragmented|Số lượng các gói dữ liệu IP đã được phân mảnh thành công
|Datagrams Failing Fragmentation|Số lượng các gói dữ liệu IP đã bị loại bỏ vì chúng cần được phân mảnh nhưng không thể
|Fragments Created|Số lượng các mảnh IP dữ liệu đã được tạo ra bởi sự phân mảnh
|Active Opens|Số lần kết nối qua các liên kết thực hiện chuyển đổi trực tiếp từ trạng thái CLOSED sang trạng thái SYN-SENT
|Passive Opens|Số lần kết nối qua các liên kết thực hiện chuyển đổi trực tiếp từ trạng thái LISTEN sang trạng thái SYN-RCVD
|Failed Connection Attempts|Số lần kết nối lại không thành công
|Reset Connections| Số lần kết nối lại
|Current Connections|Số lượng kết nối hiện tại
|Segments Received|Số segment đã nhận được
|Segments Sent|Số segment được gửi
|Segments Retransmitted|Số segment đã truyền lại
|No port| Tống số dữ liệu nhận được mà không phải từ ứng dụng ở port đích

netstat -atun | grep ...  (kiểm tra port có đang chạy hay không)

