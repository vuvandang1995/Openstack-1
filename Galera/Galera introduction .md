
# Giới thiệu Galera

Galera : Là một synchronous multi-master cluster cho MariaDB, một Multimaster Cluster dựa trên cơ chế đồng bộ hóa.

Galera Cluster tạo ra ưu thế khi có thể đọc ghi ở mọi node 

### 1. Tính năng

- Sao chép đồng bộ
- Kiến trúc active-active
- Đọc/ghi dữ liệu trên mọi cluster node
- Tự động join node
- Kết nối trực tiếp đến client

## 2. Ưu điểm

- Có tính dự phòng cao
- Khả năng đọc/ghi dữ liệu được mở rộng
- Giảm thời gian chờ của client
- Không cần slave, các node đều có thể làm master và backup cho nhau

## 3. Công nghệ sử dụng

wsrep (Write Set REPlication) là 1 API cho phép các node trong galera tái tạo dữ liệu đảm bảo đồng bộ với nhau

Các version mới nhất của wsrep và MariaDB Galera Cluster:

- Galera wsrep provider versions: 25.3.24

- MariaDB Galera Cluster versions: 10.2.18
