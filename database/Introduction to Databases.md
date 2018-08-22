# Giới thiệu về database

## I. Database
**Database** là một tập hợp dữ liệu đã được tổ chức sắp xếp. Mục đích chính của Database là để tổ chức một lượng lớn thông tin bằng việc lưu trữ, thu thập, và quản lý.

**Một database sẽ có các thành phần chính là:**

- Bảng dữ liệu: Một bảng là một ma trận dữ liệu. Một bảng trong một cơ sở dữ liệu trông giống như một bảng tính đơn giản.

- Cột: Một cột chứa cùng một kiểu dữ liệu, ví dụ như tên khách hàng.

- Hàng: Một hàng (row, entry, record) là một nhóm dữ liệu có liên quan.

- Redundancy: (có thể hiểu là dữ liệu dự phòng) Dữ liệu được lưu giữ hai lần, để làm cho hệ thống nhanh hơn.

- Primary Key: Một Primary Key (Khóa chính) là duy nhất. Một giá trị key không thể xuất hiện hai lần trong một bảng. Với một key, bạn có thể tìm thấy phần lớn trên một hàng.

- Foreign Key: Bạn tưởng tượng về Foreign Key như là cái ghim liên kết giữa hai bảng.

- Compound Key: Một Compound Key (hay composite key) là một key mà gồm nhiều cột, bởi vì một cột là không duy nhất.

- Index: Một chỉ mục trong một cơ sở dữ liệu tương tự như chỉ mục trong một cuốn sách.

- Referential Integrity: Đảm bảo rằng một giá trị Foreign Key luôn luôn trỏ tới một hàng đang tồn tại.

## II. Phân loại database

### 1. Phân loại cơ sở dữ liệu theo Mục đích:

**Cơ sở dữ liệu dạng file**: dữ liệu được lưu trữ dưới dạng các file có thể là text, ascii, .dbf. Tiêu biểu cho cơ sở dữ liệu dạng file là*.mdb Foxpro

**Cơ sở dữ liệu quan hệ**: dữ liệu được lưu trữ trong các bảng dữ liệu gọi là các thực thể, giữa các thực thể này có mối liên hệ với nhau gọi là các quan hệ, mỗi quan hệ có các thuộc tính, trong đó có một thuộc tính là khóa chính. Các hệ quản trị hỗ trợ cơ sở dữ liệu quan hệ như: MS SQL server, Oracle, MySQL...

**Cơ sở dữ liệu hướng đối tượng**: dữ liệu cũng được lưu trữ trong các bảng dữ liệu nhưng các bảng có bổ sung thêm các tính năng hướng đối tượng như lưu trữ thêm các hành vi, nhằm thể hiện hành vi của đối tượng. Mỗi bảng xem như một lớp dữ liệu, một dòng dữ liệu trong bảng là một đối tượng. Các hệ quản trị có hỗ trợ cơ sở dữ liệu hướng đối tượng như: MS SQL server, Oracle, Postgres

**Cơ sở dữ liệu bán cấu trúc**: dữ liệu được lưu dưới dạng XML, với định dạng này thông tin mô tả về đối tượng thể hiện trong các tag. Đây là cơ sở dữ liệu có nhiều ưu điểm do lưu trữ được hầu hết các loại dữ liệu khác nhau nên cơ sở dữ liệu bán cấu trúc là hướng mới trong nghiên cứu và ứng dụng.

### 2. Phân loại theo hệ điều hành

Phân loại cơ sở dữ liệu theo hệ điều hành:

**Cơ sở dữ liệu sử dụng hệ điều hành Linux**: mySQL, Mariadb

**Cở sở dữ liệu sử dụng hệ điều hành windows**: SQL Server - MSSQL

-----------------

### 3. Xu hướng có 2 loại ngôn ngữ truy vấn cơ sở dữ liệu là SQL và NO SQL

- **no sql**: nó có mấy loại như redis, cassandra, mongodb, elasticsearch

- **sql**: relation db (truyền thống), postgresql, mysql, oracle và mssql, db2 của ibm(giao diện)


#### 3.1 SQL (Structured Query Language)
SQL đã tồn tại hàng thập kỉ và có mặt gần như ở mọi app chúng ta sử dụng ngày nay. Nếu bạn triển khai app với hiệu suất tương đối thì điều đó là tuyệt vời, khỏi phải thay CSDL mới. Việc thay đổi CSDL không chỉ tốn công sức mà còn thể có nhiều rủi ro. Những loại SQL phổ biến là MySQL, Oracle, MariaDB, máy chủ MySql, Microsoft Azure.

**Điểm mạnh**

- SQL là quy chuẩn chung được chứng minh tính ổn định cao

- Tích hợp với lớp ORM như Hibernate hoặc ActiveRecord (Giải pháp mã nguồn mở, giúp đơn giản hóa ứng dụng để tương tác với cơ sở dữ liệu)

- Hỗ trợ tốt giao tác phía client

- Sử dụng query và report 

- Thị phần lớn trên thị trường

**Điểm yếu**

- Rất khó để mở rộng (kiến trúc không hướng mở rộng). 
- SQL truyền thống được xây dựng trên ý tưởng "one size fit all", application thông thường thì sử dụng tốt nhưng sẽ gặp vấn đề khi CSDL phình to
- Điều chỉnh tham số rất phức tạp và thường yêu cầu chuyên gia để cân bằng giữa hiệu năng, an toàn dữ liệu và lượng dữ liệu sử dụng

#### 3.2 NO SQL 
MongoDB, Redis , Apache Cassandra
Vì tiêu chí khác nhau nên cũng có rất nhiều hệ thống NoSQL khác nhau. 

**Loại ưu tiên tính sẵn sàng (availability) trước**
Ví dụ bạn đang nghiên cứu các dữ liệu của máy cảm biến, app của bạn ghi dữ liệu vào CSDL và thường không chỉnh sửa. Bạn muốn lưu trên cloud và không quan tâm lắm đến tính nhất quán của nó khi phân phát đến các điểm truy cập, nhiều điểm truy cập của các trung tâm dữ liệu khác ,...

Tương tác giữa app của bạn và CSDL chỉ đơn giản có "CREATE"(tạo) và "GET"(lấy). Quan trọng nhất là CSDL phải luôn chạy để nhận thêm nội dung mới và phải luôn gửi nội dung khi được yêu cầu kể cả khi nội dung đó không phải mới nhất. Các hệ thống điển hình của CSDL này là DynamoDB, Riak và Cassandra.

**Loại ưu tiên tính linh hoạt(flexibility)**
Loại này nổi tiếng hơn và có thể kể đến MongoDB và CouchDB, loại này lưu dữ liệu theo thiết kế JSON, thường được gọi là schemaless( không có schema ). CSDL loại này không có sự nhất quán về schema trong tất cả các bản ghi, điều này khiến việc quản lí schema ít bị đóng khung hơn nhưng cũng rối ren hơn. Những nhóm dev nhỏ và data đơn giản thì thường sử dụng loại này.
*(`Schema` trong SQL Server để nhóm các Database Object lại với nhau cho dễ quản lý. Có thể hiểu schema là một namespace đối với các đối tượng database, Schema mặc định của các object trong database là dbo, Một schema định nghĩa một ranh giới mà trong đó tất cả các tên là duy nhất)*

Các hệ thống khác thì mở rộng bằng việc quản lí và sắp xếp key-value tốt hơn. Redis là một thư viện nổi tiếng tạo nhiều danh sách có thể sắp xếp tiện cho xếp hạng. Tùy vào mục đích của bạn mà có thể thêm các tính năng khác cho hệ thống.

**Loại ưu tiên model data khác hoặc loại đặc biệt**
Phổ biến nhất là hệ thống được điều chỉnh phục vụ cho xử lý dạng đồ thị (graph) như Neo4j. Ví dụ khác là CSDL dạng array; SciDB sử dụng Python và R để truy cập MPP array data để nghiên cứu khoa học. Accumulo là sự dung hợp giữa kiểu CSDL cột rộng của Cassandra và BigTable nhưng tập trung vào từng ô hơn. Hệ thống như etcd phân phối dữ CSDL tập trng vào lưu trữ cấu hình data cho các dịch vụ khác. Elasticsearch là hệ thống nổi tiếng ứng dụng tìm kiếm text trong app.

**Ưu điểm**
- Thuật toán eventual-consistency đảm bảo tính sẵn sàng của CSDL

- Khả năng mở rộng tốt hơn SQL truyền thống

- Rất nhiều hệ thống NoSQL  được tối ưu để hỗ trợ data không quan hệ ví dụ như log message, XML và JSON cũng như document không có kiến trúc cụ thể nên bạn không cần quan tâm schema khi viết mà chỉ cần xem schema khi đọc.

**Nhược điểm**

- Hệ thống dạng này không hỗ trợ giao tác (ACID)

- OLAP query yêu cầu lượng code lớn để thực hiện và ở dạng này CSDL dữ liệu không nhất quán, do đó việc truy suất dữ liệu sẽ giảm khi CSDL phình to.


----
```
- Dự phòng : Do một dạng dữ liệu có cấu trúc, dự phòng dữ liệu trong cơ sở dữ liệu SQL là quan trọng. Bản chất quan hệ của cơ sở dữ liệu SQL cho phép nó tránh được sự dư thừa ở cấp độ rất sơ cấp. Ngược lại, dự phòng dữ liệu là một hạn chế lớn trong cơ sở dữ liệu NoSQL. 

- Tính linh hoạt : Khả năng lưu trữ các loại dữ liệu khác nhau với nhau là một thuộc tính sở hữu bởi cơ sở dữ liệu NoSQL. Vì cơ sở dữ liệu NoSQL là tài liệu dựa trên nó có thể kết hợp bất kỳ loại dữ liệu nào trong tài liệu của nó. 

- Tốc độ : Với khả năng truy cập dễ dàng và cấu trúc linh hoạt, NoSQL thành công trên cơ sở dữ liệu SQL trong bộ phận tốc độ. Nó chỉ đơn giản có nghĩa là NoSQL mất ít thời gian hơn để truy cập một phần dữ liệu hơn SQL. 

- Bộ nhớ trên đám mây : Lưu trữ dữ liệu trên đám mây yêu cầu dữ liệu phải được lưu trữ trên nhiều máy chủ. Để nó diễn ra, cơ sở dữ liệu bạn chọn nên có khả năng mở rộng cao. Các cơ sở dữ liệu NoSQL được thiết kế để có thể mở rộng trên nhiều máy chủ.
```
-----

## III. Cơ chế SQL và NO SQL

**1. SQL**

lưu dữ liệu theo các bảng và khi truy vấn cũng theo các bảng, các bảng sử dụng các `khóa chính` và `khóa ngoại`, các khóa nội bảng này liên kết với khóa ngoại bảng kia để cho dữ liệu nhất quán, đồng bộ.
- Việc mở rộng database khó khăn do các bảng sử dụng khóa chính và khóa ngoại liên kết với nhau, khi thêm bảng  bảng cần phải liên kết các key chính và key ngoại rất phức tạp hầu hết là phải xây dựng lại, xóa database nếu không hiểu rõ database đó có thể sẽ gây hỏng database do các bảng liên kết với nhau, vì vậy cần khảo sát dữ liệu trước khi xây dựng.

<img src="https://i.imgur.com/GI61Itj.png">

- Các cột ProductID và VendorID trong bảng Purchasing:


**2. NO SQL**

Lưu dữ liệu theo các dòng timestamp và khi truy vấn sẽ truy vấn theo timestamp, sử dụng `key` và `value` và phân loại theo dạng thời gian thực (tính theo nano giây) kèm theo đó là thông số về ram, cpu..., và nó lưu trên ram cache đây là ưu điểm để truy suất dữ liệu nhanh, sau 1 thời gian tùy theo loại no sql quy định của người lập trình thì các dữ liệu cũ sẽ được đẩy xuống và lưu ở disk (trở thành dữ liệu cũ). 

- Việc mở rộng database dễ dàng do không có sự ràng buộc giữa các bảng.


Các công cụ kết nối tới database trên window là navicat
