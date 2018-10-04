
Galera cần khởi động 1 node cluster giống như 1 điểm gốc trước khi các node còn lại có thể joint vào cụm cluster. Tiến trình như vậy gọi là cluster bootstrap. Bootstrao là bước đầu tiên để quảng bá 1 node primary database trước khi các node khác xem node gốc làm điểm tham chiếu để đồng bộ dữ liệu.

Khi Galera khởi động bằng bootstrap trên 1 node, node được bootstrap đó sẽ đổi trạng thái thành primary (kiểm tra bằng lệnh wsrep_cluster_status). Các node còn lại sẽ chỉ cần start bằng lệnh bình thường và sẽ tự động  tìm kiếm các thành phần chính hiện có trong cluster và joint vào thành 1 cluster. Đồng bộ hóa dữ liệu sau đó sẽ thông qua IST ( incremental state transfer ) hoặc SST ( snapshot state transfer ).


