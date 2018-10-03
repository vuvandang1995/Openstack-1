# Tìm hiểu về Galera 

Galera : Là một Multimaster Cluster dựa trên cơ chế đồng bộ hóa.

Galera Cluster tạo ra ưu thế khi có thể đọc ghi ở mọi node

Cấu hình và cài đặt Galera Cluster</a>

+ Bước 1: Thực hiện cài đặt MariaDB trên cả 3 node DB01, DB02 và DB03 như sau:

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

				10.10.10.10 lb03
				10.10.10.20 db01
				10.10.10.30 db02
				10.10.10.40 db03

		+ Cài đặt MariaDB trên lần lượt các node db01, db02 và db03

				yum install mariadb-server rsync

+ Bước 2: Cấu hình cài đặt, tạo galera cluster cho 3 node db01, db02 và db03:

		+ Thực hiện trên node db01:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db01,db02,db03"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="10.10.10.20"
			# this server name, change for each server
			wsrep_node_name="db01"

			wsrep_sst_method=rsync

		+ Thực hiện trên node db02:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db01,db02,db03"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="10.10.10.30"
			# this server name, change for each server
			wsrep_node_name="db02"

			wsrep_sst_method=rsync

		+ Thực hiện trên node db03:

				vi /etc/my.cnf.d/server.cnf

		thêm nội dung sau vào file trong phần mục [galera] và lưu lại:

			# Mandatory settings
			wsrep_on=ON
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so

			#add your node ips here
			wsrep_cluster_address="gcomm://db01,db02,db03"
			binlog_format=row
			default_storage_engine=InnoDB
			innodb_autoinc_lock_mode=2
			#Cluster name
			wsrep_cluster_name="galera_cluster"
			# Allow server to accept connections on all interfaces.

			bind-address=0.0.0.0

			# this server ip, change for each server
			wsrep_node_address="10.10.10.40"
			# this server name, change for each server
			wsrep_node_name="db03"

			wsrep_sst_method=rsync

+ Bước 3: Tắt SeLinux và thực hiện mở port cho 3 node db01, db02 và db03:

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

		- Câu lệnh này chỉ thực hiện trên một node, giả sử ta chạy trên node db01:

				galera_new_cluster

			sau khi câu lệnh thực hiện chạy xong,  thực hiện chạy lệnh sau trên cả 2 node db02 và db03 để tiến hành đồng bộ:

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

			Con số *3* chỉ số node đã join vào trong cụm galera.
