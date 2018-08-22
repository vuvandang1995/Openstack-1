# Cách cài đặt mysql

-Cài đặt wget

yum install wget -y

- Lấy repo từ trang chủ

wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

- Cài đặt Repo

rpm -ivh mysql-community-release-el7-5.noarch.rpm

B1: Chuyển sang quyền root :

```
sudo su
```

B2: Cập nhật các Repository:

```
yum update -y
```

B3: Cài đặt mysql-server bằng lệnh:

```
yum install mysql-server -y 
```

B4: Khởi động mysql

```
systemctl start mysql
```

B5: Mở mysql

```
systemctl enable mysql
```

B6: Kiểm tra lại trạng thái của mysql

```
systemctl status mysql
```
B7: Truy cập mysql
```
mysql -u root -p
```
mặc định password không có gì (trên centos)

-----

## Các lệnh cơ bản trong Mysql

### Các lệnh thao tác tạo, sửa , xóa database

**1.Tạo database:**
```
CREATE DATABASE 'ten database';  (Ví dụ: create database bigdata;
   ```
   
**2.Chọn 1 database để sử dụng:**
```
USE 'tên database';   (ví dụ: use bigdata;)
```

**3.Xóa một database:**

```
DROP DATABASE 'tên database';  (ví dụ: drop database bigdata;)
```

**4.Hiển thị các database:**

```
SHOW DATABASES; 
```

**5.Hiển thị chi tiết trong bảng**

`select * from 'tên bảng';`

---
###  Các lệnh liên quan đến bảng:

**1.Hiển thị thông tin các tables trong database nào đó đang truy cập:**
```
SHOW TABLES;
```
**2.Tạo 1 bảng:**
```
CREATE TABLE ten_bang(
   kieu_du_lieu cot1,
   kieu_du_lieu cot2,
   kieu_du_lieu cot3,
   .....
   kieu_du_lieu cotN,
   PRIMARY KEY( mot hoac nhieu cot )
);

Ví dụ:
SQL> CREATE TABLE an(
   ID   INT              NOT NULL,
   TEN VARCHAR (20)     NOT NULL,
   TUOI  INT              NOT NULL,
   KHOAHOC  CHAR (25) ,
   HOCPHI   DECIMAL (18, 2),       
   PRIMARY KEY (ID)
);
```

<img src="https://i.imgur.com/MjoiyWh.png">

**3. Hiển thị rút gọn các trường fied của 1 table**
```
show columns from 'tên table'
```

**4.Nhập data vào table:**
```
INSERT INTO [TEN TABLE](cột 1, cột 2,...) VALUES (gicột 1,gi côt 2...);
```

Ví dụ:
insert into an (id,ten,tuoi) values (1,'nam',10);  lưu ý với tên thì phải có dấu nháy đơn ''

<img src="https://i.imgur.com/86505Uj.png">

**5.Lấy thông tin từ bảng:**
```
SELECT 'Tên cột muốn hiển thị' FROM 'Tên bảng' [WHERE 'Điều kiện'];
```
**6.Cập nhật thông tin cho các bảng:**
```
UPDATE [Tên TABLE] SET [cột muốn sửa] [Where 'Điều kiện'];
```
Ví dụ:

update an set khoahoc = 'ok' where id = 2;

<img src="https://i.imgur.com/LA485dS.png">

**7.Xóa TABLE :**

```
DROP Table 'Tên table';
```
---
### Các lệnh về User: 

**1.Tạo user:**

```
 CREATE USER 'tên user'@'tên miền';  (ví dụ: create user pac@google;)
 hoặc create user pac@localhost identified by ‘password’ (đặt luôn password)
```

**2.Đặt password cho user:**

```
SET PASSWORD FOR 'Tên user'=PASSWORD(''password''); ( ví dụ: SET PASSWORD FOR pac@google=PASSWORD('123456')
```

**3.Gán quyền cho user:**

```
 GRANT 'quyền' ON 'tên database'.'tên table' TO 'user'@'localhost';
```

```
SHOW GRANTS FOR pac6@'%'; (xem quyền gán cho user)
```

- các  quyền  hạn trong mysql:

```
   ALL PRIVILEGES- tất cả quyền trên hệ thống
   CREATE- cho phép tạo database và table
   DROP- cho phép xóa database hoặc table
   DELETE- cho phép xóa các record trong table
   INSERT- cho phép thêm record trong table
   SELECT-cho phép thực hiện lệnh SELECT để query dữ liệu
   UPDATE- cho phép cập nhật record trong table
   GRANT OPTION- cho phép thực hiện tác vụ phân quyền cho user khác
   (GRANT GRANT OPTION .....)
```
**4.Xem thông tin các user**

```
select user,host from mysql.user; (xem trường user và host từ cơ sở dữ liệu tên là mysql với table là user)
```

- Để quyền có hiệu lực cần thực hiện lệnh sau:

```
FLUSH PRIVILEGES;
```

-Xóa  quyền :

```
 REVOKE [quyền] ON [tên database].[tên table] FROM '[user]'@'localhost';
 example :
    REVOKE INSERT ON *.* FROM jeffrey@localhost;
```


|Privilege	|Meaning
|-----------|--------------------------------------------
|ALL [PRIVILEGES]	|  Tất cả các quyền trừ GRANT OPTION
|ALTER	| cho phép thêm một cột mới trong một bảng đang tồn tại (Cho phép thay đổi cấu trúc của bảng hiện tại)
|CREATE|	cho phép tạo bảng
|CREATE TEMPORARY TABLES|	Cho phép tạo ra các bảng tạm thời khi bạn muốn lưu giữ dữ liệu tạm thời, Các bảng tạm sẽ chỉ tồn tại khi session còn sống
|DELETE	|cho phép xóa các hàng từ một bảng
|DROP	|cho phép xóa một bảng và tất cả dữ liệu
|EXECUTE|	Cho phép thực thi câu lệnh
|FILE	|Cho phép nhập dữ liệu và xuất khẩu dữ liệu vào tập tin
|INDEX	|Cho phép tạo hoặc xóa các chỉ số
|INSERT|	cho phép INSERT
|LOCK TABLES|	Cho phép khóa bảng 
|PROCESS	|cho phép xem các tiến trình
|REFERENCES|	là quyền khai báo field của bảng A tham chiếu đến field của bảng B (tạo khóa chính khóa ngoại)
|RELOAD	| cho phép thực thi các quyền đã gán cho các user (flush)
|REPLICATION CLIENT	|Cho phép sử dụng slaves/masters (REPLICATION là một quá trình cho phép bạn dễ dàng duy trì nhiều bản sao của dữ liệu bằng cách sao chép tự động từ một master tạo ra một cơ sở dữ liệu slave
|REPLICATION SLAVE	|cấp quyền cho các slave, cần thiết cho slaves nhân rộng
|SELECT	|cho phép  đọc dữ liệu trong database
|SHOW DATABASES|	hiển thị các databases
|SHUTDOWN	|cho phép tắt mysql server
|SUPER	|Cho phép kết nối với quyền quản trị cao nhất
|UPDATE	|Cho phép thay đổi dữ liệu
|USAGE	|gán toàn bộ các quyền của user là "không được phép". Quyền này thường được gán cho các account không có quyền global trên hệ thống. Thường sau khi gán quyền USAGE, quản trị viên của server sẽ tiếp tục gán một vài quyền nhất định cho account trên một số CSDL nhất định
|GRANT OPTION	| Cho phép thêm người sử dụng và đặc quyền 

**5.Liên kết + ràng buộc primary key và foreign key**
Ví dụ:
```
CREATE TABLE an(
   ID   INT              NOT NULL,
   TEN VARCHAR (20)     NOT NULL,
   TUOI  INT              NOT NULL,
   KHOAHOC  CHAR (25) ,
   HOCPHI   DECIMAL (18, 2),       
   PRIMARY KEY (ID)
);

create table hoc(
       ngay int not nul,
       thang int references an(id), (liên kết ràng buộc với mục id của an)
       primary key (ngay)
       );

ALTER TABLE hoc ADD FOREIGN KEY (thang) REFERENCES an(id);
```

---

**6.Backup dữ liệu:**

```
mysqldump -u root -p [tendatabase] > [name.sql];
```
    
ví dụ:
mysqldump -u root -p bigdata  > name.sql;  (lệnh này không chạy trong terminal của mysql mà chạy ở ngoài terminal linux)

nó sẽ tạo ra 1 file đuôi .sql 

- Backup thông qua MySQL Dump - `mysqldump`
 Đây là các CMD SHELL trên Linux

-Backup toàn bộ DB
```
mysqldump -u [uname] -p [tên db] > [file backup].sql
```
- Backup toàn bộ DB
```
mysqldump -u [uname] -p --all-databases > [file backup].sql
```
- Backup tables chỉ định
```
mysqldump -u [uname] -p [tên db] [table1] [table2] > [file backup].sql
```
- Backup + nén sang dạng gzip
```
mysqldump -u [uname] -p [tên db] | gzip >  [file backup].sql.gz
```

- Restore file backup
```
mysql -u [User] -p < [file backup].sql
```

