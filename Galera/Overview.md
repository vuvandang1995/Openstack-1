# Tìm hiểu về Galera 

Galera : Là một synchronous multi-master cluster cho MariaDB, một Multimaster Cluster dựa trên cơ chế đồng bộ hóa.

Galera Cluster tạo ra ưu thế khi có thể đọc ghi ở mọi node 

## Cấu hình và cài đặt Galera Cluster

### 3 node:
db2: 192.168.40.62
db3: 192.168.40.63
db4: 192.168.40.64

+ Bước 1: Thực hiện cài đặt MariaDB trên cả 3 node db2, db3 và db4 như sau:

			vi /etc/yum.repos.d/MariaDB.repo

		thêm nội dung sau vào file, sau đó lưu lại:

			[mariadb]
			name = MariaDB
			baseurl = http://yum.mariadb.org/10.2/centos7-amd64
			gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
			gpgcheck=1


		+ Thực hiện trỏ host (thêm thông tin các host) vào file /etc/hosts trên cả 3 node:

				vi /etc/hosts

			thêm nội dung dưới đây vào file và sau đó lưu lại:

				
				192.168.40.62 db2
				192.168.40.63 db3
				192.168.40.64 db4

		+ Cài đặt MariaDB trên lần lượt các node db2, db3 và db4

				yum install mariadb-server rsync

+ Bước 2: Cấu hình cài đặt, tạo galera cluster cho 3 node db2, db3 và db4:

		+ Thực hiện trên node db2:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db2,db3,db4"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="192.168.40.62"
			# this server name, change for each server
			wsrep_node_name="db2"

			wsrep_sst_method=rsync

		+ Thực hiện trên node db3:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db2,db3,db4"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="192.168.40.63"
			# this server name, change for each server
			wsrep_node_name="db3"

			wsrep_sst_method=rsync

		+ Thực hiện trên node db4:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db2,db3,db4"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="192.168.40.64"
			# this server name, change for each server
			wsrep_node_name="db4"

			wsrep_sst_method=rsync

+ Bước 3: Tắt SeLinux và thực hiện mở port cho 3 node db2, db3 và db4:

		- Mở các port cho phép thực hiện truy vấn database và liên hệ giữa các node trong cụm galera:

				firewall-cmd --permanent --add-port=3306/tcp
				firewall-cmd --permanent --add-port=4567/tcp

		- Mở port cho phép rsync đồng bộ dữ liệu với nhau:

				firewall-cmd --permanent --add-port=873/tcp

		- Mở ports quan trọng khác:

				firewall-cmd --permanent --add-port=4444/tcp
				firewall-cmd --permanent --add-port=9200/tcp

		- Nạp lại chính sách firewall, ...:

				firewall-cmd --reload

+ Bước 4: Khởi tạo galera cluster:

		- Câu lệnh này chỉ thực hiện trên một node, giả sử ta chạy trên node db2:

				galera_new_cluster

			sau khi câu lệnh thực hiện chạy xong,  thực hiện chạy lệnh sau trên cả 2 node db3 và db4 để tiến hành đồng bộ:

				systemctl start mysql

		- Sau khi câu lệnh chạy hoàn tất, chạy câu lệnh sau để kiểm tra đã thực hiện cấu hình thành công hay chưa:

				mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"

			kết quả như sau:

				Enter password:
				+--------------------+-------+
				| Variable_name      | Value |
				+--------------------+-------+
				| wsrep_cluster_size | 3     |
				+--------------------+-------+

			Con số 3 chỉ số node đã join vào trong cụm galera.
	- Cấp quyền truy cập database (yêu cầu trên load balancer đã được cài đặt maxscale):

			# mysql -u root -p
			Enter password: 

	- Tạo người sử dụng:

			CREATE USER 'maxscale'@'%' IDENTIFIED BY 'password';
			GRANT SELECT ON mysql.db TO 'maxscale'@'%';
			GRANT SELECT ON mysql.user TO 'maxscale'@'%';
			GRANT SHOW DATABASES ON *.* TO 'maxscale'@'%';
	Sau đó ta có thể xem thông tin user maxscale trên 3 db
