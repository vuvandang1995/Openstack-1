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


