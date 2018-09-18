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
