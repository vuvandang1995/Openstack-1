# Cơ chế hoạt động Bootstrap MySQL, MariaDB Galera Cluster

Galera cần khởi động 1 node cluster giống như 1 điểm gốc trước khi các node còn lại có thể joint vào cụm cluster. Tiến trình như vậy gọi là cluster bootstrap. Bootstrao là bước đầu tiên để quảng bá 1 node primary database trước khi các node khác xem node gốc làm điểm tham chiếu để đồng bộ dữ liệu.

Khi Galera khởi động bằng bootstrap trên 1 node, node được bootstrap đó sẽ đổi trạng thái thành primary (kiểm tra bằng lệnh wsrep_cluster_status). Các node còn lại sẽ chỉ cần start bằng lệnh bình thường và sẽ tự động  tìm kiếm các thành phần chính hiện có trong cluster và joint vào thành 1 cluster. Đồng bộ hóa dữ liệu sau đó sẽ thông qua IST ( incremental state transfer ) hoặc SST ( snapshot state transfer ).

Chỉ nên sử dụng bootstrap cluster nếu muốn khởi động 1 cluster mới hoặc trong cluster không có node nào là primary state.
Việc sử dụng bootstrap cần thận trọng nếu không các node trong cluster sẽ bị phân tách hoặc mất dữ liệu.

Ví dụ sau minh họa khi sử dụng bootstrap cho 3 node dựa trên node state (wsrep_local_state_comment) và cluster state (wsrep_cluster_status):
<img src="https://i.imgur.com/1yfK6Nh.png">
<img src="https://i.imgur.com/n0VgmuQ.png">

## Cách start galera cluster:

- Trên MySQL Galera Cluster (Codership):
```
$ service mysql bootstrap                     # sysvinit
$ galera_new_cluster                          # systemd
$ mysqld_safe --wsrep-new-cluster             # command line
```
- Trên Percona XtraDB Cluster (Percona):
```
$ service mysql bootstrap-pxc                 # sysvinit
$ systemctl start mysql@bootstrap.service     # systemd
```
- Trên MariaDB Galera Cluster (MariaDB):
```
$ service mysql bootstrap                     # sysvinit
$ service mysql start --wsrep-new-cluster     # sysvinit
$ galera_new_cluster                          # systemd
$ mysqld_safe --wsrep-new-cluster             # command line
```

Sau khi node đầu tiên lên chúng ta restart lại mysql:
```
$ service mysql start
$ systemctl start mysql
```

Bây giờ node mới sẽ kết nới tới các node trong cluster tạo thành 1 cluster

## Cảnh báo: Không bao giờ được sử dụng bootstrap khi muốn kết nối lại 1 node tới 1 cluster đã có sẵn, và không bao giờ được sử dụng bootstrap trên nhiều node, chỉ được sử dụng trên 1 node.

## Trường an toàn khi khởi động bootstrap

Galera từ phiên bản 3.19 sẽ có 1 trường là **safe_to_bootstrap** bên trong `grastate.dat`, nó là 1 trường điều kiện ngăn chặn các lựa chon không an toàn bằng cách theo dõi thứ tự các node đang tắt, node tắt cuối cùng sẽ được đánh dấu là **Safe-to-Bootstrap** và chỉ được bootstrap trên node đó, còn các node còn lại sẽ bị đánh dấu là không an toàn để bootstrap.

- File `/var/lib/mysql/grastate.dat`:
```
# GALERA saved state
version: 2.1
uuid:    8bcf4a34-aedb-14e5-bcc3-d3e36277729f
seqno:   2575
safe_to_bootstrap: 0
```
- Khi bootstrap 1 cluster mới, Galera sẽ từ chối khởi động node đầu tiên được đánh dấu là không an toàn để bootstrap, ta sẽ thấy trong logs:
```
“It may not be safe to bootstrap the cluster from this node. It was not the last one to leave the cluster and may not contain all the updates.

To force cluster bootstrap with this node, edit the grastate.dat file manually and set safe_to_bootstrap to 1 .”
```

Trong trường hợp sự cố các node bị tắt đột ngột hoặc ngoài ý muốn thì các node sẽ có trường **safe_to_bootstrap: 0**, và sẽ tham vấn bộ lưu trữ **InnoDB** mục đích để các node thực hiện những thao tác, giao dịch cuối cùng trong cluster. 

- Điều này được thực hiện bằng cách thực hiện lệnh sau trên mỗi node: `mysqld --wsrep-recover`
```
$ mysqld --wsrep-recover
...
2016-11-18 01:42:15 36311 [Note] InnoDB: Database was not shutdown normally!
2016-11-18 01:42:15 36311 [Note] InnoDB: Starting crash recovery.
...
2016-11-18 01:42:16 36311 [Note] WSREP: Recovered position: 8bcf4a34-aedb-14e5-bcc3-d3e36277729f:114428
...
```

Ký tự sau UUID là chuỗi cần tìm cho node sẽ bootstrap, chọn node có số cao nhất và chỉnh sửa `grastate.dat` của nó để đặt “safe_to_bootstrap: 1”  như ví dụ sau:
```
cat /var/lib/mysql/grastate.dat
# GALERA saved state
version: 2.1
uuid:    8bcf4a34-aedb-14e5-bcc3-d3e36277729f
seqno:   -1
safe_to_bootstrap: 1
```
Sau đó ta có thể bootstrap trên node đã chọn

Trong trường hợp các node bị tách khởi cluster bì một lý do ngoài ý muốn, lúc đó ta cần chọn 1 node và quảng bá nó để trở thành 1 primary.

- Để xác định nút nào cần được khởi động, hãy so sánh giá trị **wsrep_last_committed** cao nhất trên tất cả các nút DB, truy cập vào database:

```
mysql -u root -p

node1> SHOW STATUS LIKE 'wsrep_%';
+----------------------+-------------+
| Variable_name        | Value       |
+----------------------+-------------+
| wsrep_last_committed | 10032       |
...
| wsrep_cluster_status | non-Primary |
+----------------------+-------------+
```
```
node2> SHOW STATUS LIKE 'wsrep_%';
+----------------------+-------------+
| Variable_name        | Value       |
+----------------------+-------------+
| wsrep_last_committed | 10348       |
...
| wsrep_cluster_status | non-Primary |
+----------------------+-------------+
```
```
node3> SHOW STATUS LIKE 'wsrep_%';
+----------------------+-------------+
| Variable_name        | Value       |
+----------------------+-------------+
| wsrep_last_committed |   997       |
...
| wsrep_cluster_status | non-Primary |
+----------------------+-------------+
```
Trong trường hợp này, tất cả các nút Galera đã được bắt đầu, do đó bạn không nhất thiết phải khởi động lại cụm đó. Chúng ta chỉ cần quảng bá node2 trở thành một primary Component

`node2> SET GLOBAL wsrep_provider_options="pc.bootstrap=1";`

Các node còn lại sau đó sẽ kết nối lại với Primary Component (node2) và đồng bộ hóa dữ liệu của chúng dựa trên nút này.
