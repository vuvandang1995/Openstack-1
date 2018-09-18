# Thông số giám sát hệ thống



## 1. Ram

**RAM** ( Randon Access Memory) - Là bộ nhớ tạm thời dữ liệu được lưu trữ trên RAM đều sẽ bị mất ngay sau khi máy tính bạn tắt. Ram là bộ nhớ truy cập ngẫu nhiên, CPU có thể truy cập vào một địa chỉ nhớ bất kỳ mà không cần phải truy cập một cách tuần tự, điều này cho phép CPU đọc và ghi vào RAM vớ tốc độ nhanh hơn. Tốc độ RAM hiện đại vượt qua tốc độ 15000 MB/s so với tốc độ ổ cứng tốt nhất là 1000 MB/s.

### 1.1. Đặc điểm

- Khi một chương trình hoạt động (chẳng hạn như hệ điều hành, hay các ứng dụng) hoặc mở một tập tin (chẳng hạn như video, hình ảnh, nhạc, tài liệu...), nó sẽ được load tạm thời từ ổ đĩa cứng vào RAM.

- Nếu RAM bị hết, hệ điều hành của bạn sẽ bắt đầu “dump” một số chương trình mở và các tập tin thành paging file (chạy chương trình trên ổ cứng nếu ít ram). Nếu paging file được lưu trữ quá nhiều sẽ khiến ổ đĩa cứng của bạn ngày một chậm dần. Do đó thay vì chạy mọi thứ trên RAM, một phần sẽ được truy cập từ ổ đĩa cứng.

### 1.2. Phân loại RAM

RAM thường có 3 loại: DDR 1, 2, và 3

- RAM DDR1 thường có xung nhịp từ 266MHz tới 400MHz.

- RAM DDR2 và DDR3 (loại mới nhất) thường có xung nghiệp từ 400 - 800 MHz và từ 800 MHz - 1.6 GHz.

- Giá cả giữa DDR1, DDR2 và DDR3 chênh lệch nhau rất nhiều.

### 1.3. Cách kiểm tra các thông số RAM trên Linux
```
#free
#free -m
#free -h
```

- total : Tổng RAM vật lý,(không bao gồm một bit nhỏ mà kernel thường dự trữ cho chính nó lúc khởi động).
- used : Bộ nhớ được sử dụng bởi hệ điều hành.
- free : Bộ nhớ đang trống, chưa được sử dụng.
- shared/buffers/cached : Dung lượng sử dụng cho bộ nhớ đệm
- swap : cung cấp thông tin về việc sử dụng không gian hoán đổi (tức là các nội dung bộ nhớ đã tạm thời được chuyển sang đĩa).

## 2. Tìm hiểu về CPUs

### 2.1 Định nghĩa CPU

- CPU viết tắt của chữ Central Processing Unit (tiếng Anh): bộ xử lý trung tâm, là các mạch điện tử trong một máy tính, thực hiện các câu lệnh của chương trình máy tính bằng cách thực hiện các phép tính số học, logic, so sánh và các hoạt động nhập/xuất dữ liệu (I/O) cơ bản do mã lệnh chỉ ra.
- CPU có nhiều kiểu dáng khác nhau. Ở hình thức đơn giản nhất, CPU là một con chip với vài chục chân. Phức tạp hơn, CPU được ráp sẵn trong các bộ mạch với hàng trăm con chip khác.
- CPU là một mạch xử lý dữ liệu theo chương trình được thiết lập trước. Nó là một mạch tích hợp phức tạp gồm hàng triệu transistor.

### 2.2 Vai trò của CPU

- **CPU** là bộ vi sử lý mà  tại đó mọi thông tin, thao tác, dữ liệu sẽ được tính toán kỹ lưỡng và đưa ra lệnh điều khiển mọi hoạt động của máy tính.

**Các thành phần trong CPU:**

<img src="https://i.imgur.com/01bPis1.jpg">

- **Bộ điều khiển**(CU-Control Unit): Là các vi xử lí có nhiệm vụ thông dịch các lệnh của chương trình và điều khiển hoạt động xử lí, được điều tiết chính xác bởi xung nhịp đồng hồ hệ thống. Mạch xung nhịp đồng hồ hệ thống dùng để đồng bộ các thao tác xử lí trong và ngoài CPU theo các khoảng thời gian không đổi.Khoảng thời gian chờ giữa hai xung gọi là chu kỳ xung nhịp.Tốc độ theo đó xung nhịp hệ thống tạo ra các xung tín hiệu chuẩn thời gian gọi là tốc độ xung nhịp – tốc độ đồng hồ tính bằng triệu đơn vị mỗi giây-Mhz. Thanh ghi là phần tử nhớ tạm trong bộ vi xử lý dùng lưu dữ liệu và địa chỉ nhớ trong máy khi đang thực hiện tác vụ với chúng.

- **Bộ số học-logic**(ALU-Arithmetic Logic Unit):  Các con số toán học và logic sẽ được tính toán kỹ càng và đưa ra kết quả cho các quá trình xử lý kế tiếp. Theo tên gọi,đơn vị này dùng để thực hiện các phép tính số học( +,-,/ )hay các phép tính logic (so sánh lớn hơn,nhỏ hơn…)

- **Các thanh ghi**: Là các bộ nhớ có dung lượng nhỏ nhưng tốc độ truy cập rất cao, nằm ngay trong CPU, dùng để lưu trữ tạm thời các toán hạng, kết quả tính toán, địa chỉ các ô nhớ hoặc thông tin điều khiển. Mỗi thanh ghi có một chức năng cụ thể. Thanh ghi quan trọng nhất là bộ đếm chương trình (PC - Program Counter) chỉ đến lệnh sẽ thi hành tiếp theo.

