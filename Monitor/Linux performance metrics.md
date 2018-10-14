# Thông số metric chủ yếu sử dụng monitor

## 1. Processor metrics

- **CPU utilization**:  Mô tả mức sử dụng tổng thể của bộ vi xử lý, nếu việc sử dụng CPU khiến CPU utilization đạt ngưỡng 80% trong một thời gian sẽ gây ra hiện tượng thắt cổ chai gây chậm hệ thống.

- **User time**: Mô tả tỉ lệ phần trăm CPU sử dụng cho các tiến trình người dùng. Giá trị cao là giá trị mong muốn.

- **System time**: Tỉ lệ phần trăm CPU dành cho hệ điều hành kernel bao gồm IRQ và softirq time. Các giá trị này nếu cao cũng sẽ gây hiện tượng nghẽn cổ chai trong mạng và các driver stack. Một hệ thống thông thường tốn ít CPU cho kernel.

- **Waiting**: Tổng thời gian CPU đã sử dụng cho 1 hoạt động I/O. Một hệ thống không nên có quá nhiều time cho các hoạt động của I/O, nếu không thì 

- **Idle time**: Số phần trăm CPU hệ thống chưa sử dụng và chờ cho tiến trình kế tiếp.

- **Nice time**: Số phần trăm CPU sử dụng cho việc tính toán độ ưu tiên cho các tiến trình.

- **Load average**: Giá trị tải trung bình, số lượng process cần tài nguyên tính toán của CPU tại thời điểm nhất định, giá trị load average càng tăng tức là đang có càng nhiều số tiến trình cần xử lý.

- **Runable processes**: Là 1 tiến trình TASK_RUNNING, một chương trình thực thi, và nó chứa danh sách các tiến trình có thể chạy cho CPU, khuyến cáo không nên vượt quá 10 lần số lượng bộ vi xử lý vật lý nếu không có thể gây nghẽn cổ chai.

-  **Blocked**: Các tiến trình đang bị chặn và không thể thực thi và điều này có thể dẫn tới việc nghẽ cổ chai khi nó hoạt động trở lại.

- **Context switch**: là quá trình lưu trữ trạng thái của một tiến trình hoặc của một luồng, để nó có thể được khôi phục và thực hiện được tiếp tục từ cùng một điểm sau đó. Điều này cho phép nhiều tiến trình chia sẻ một CPU, và là một tính năng thiết yếu của một hệ điều hành đa nhiệm.

- **Interrupts** Gồm ngắt cứng (hard interrupts) và ngắt mềm (soft interrupts), giá trị ngắt cứng cao có thể ảnh hưởng tới hiệu suất làm việc của hệ thống. Nó còn có thể do xung đồng hồ CPU gây ra

## 2. Memory metrics

- **Free memory** Số lượng Ram còn trống

- **Swap usage** Số lượng ram ảo hay nói cách khác là dung lượng chuyển từ ram sang chạy trên ổ đĩa

- **Buffer and cache**: là bộ nhớ đệm sử dụng cho việc truy xuất dữ liệu nhanh hơn. Được cấp phát dưới dạng file system hoặc block devide

- **Slabs**: Mô tả việc sử dụng bộ nhớ kernel. Lưu ý các Kernel không thể được chạy trên ổ cứng.

- **Active versus inactive memory**: Cung cấp thông tin về việc sử dụng bộ nhớ hệ thống đang hoạt động hay không hoạt động.

## 3. Network interface metrics

- **Packets received and sent**: Số lượng packet nhận và gửi

- **Bytes received and sent**: Số lượng byte nhận và gửi cho 1 giao diện mạng

- **Collisions per second**: Số va chạm của các packet trên 1 giây trên 1 giao diện mạng kết nối tới, nếu giá trị càng cao càng cho thấy tốc độ mạng bị chậm hoặc có thể bị treo.

- **Packets dropped**: số packet bị mất

- **Overruns**: Chỉ ra rằng vượt qua dung lượng bộ đệm trên giao diện mạng, khuyến cáo sử dụng kết hợp với drop để xác định nghẽn bộ đệm mạng.

- **Errors**: số frames bị lỗi trên đường truyền

## 4. Block device metrics

- **Iowait**: Thời gian CPU sử dụng cho 1 I/O, giá trị càng cao càng gây nghẽn

- **Average queue length**: Số lượng các I/O đang chờ xử lý, 1 disk có khoảng 2 đến 3 giá trị là tối ưu, giá trị cao quá cũng có thể gây nghẽn disk I/O 

- **Average wait**: Thời gian chờ trung bình cho một xử lý yêu cầu cho 1 I/O

- **Transfers per second**: Số lượng  I/O hoạt động trên 1 giây

- **Blocks read/write per second**: Số lần đọc ghi mỗi giây cho block 1024 bytes

- **Kilobytes per second read/write**: Dung lượng dữ liệu được đọc ghi truyền tới block device hoặc từ các block devide truyền đến
