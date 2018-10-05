# Tìm hiểu về HA Proxy

[I. Một số thuật ngữ](#1)

[1. Access Control List (ACL)](#2)

[2. Backend](#3)

[3. Frontend](#4)

[II. Các dạng của Load Balancing](#5)

[1. Layer 4 Load Balancing](#6)

[2. Layer 7 Load Balancing](#7)

[3. Các thuật toán Cân bằng tải](#8)

[4. Sticky Session](#9)

[5. Health Check](#10)

[III. Mô hình kết hợp với keepalive](#11)


**HAProxy - High Availability Proxy**: Mục đích sử dụng là cân bằng tải TCP/HTTP

- Phần mềm mã nguồn mở Load Balancer và proxying TCP/HTTP chạy trên Linux, Solaris và FreeBSD

- Được dùng nhằm tăng hiệu năng và đáp ứng sự phân tán khối lượng công việc trên các server (web, ứng dụng, database...)

- Được sử dụng trong rất nhiều môi trường cấu hình cao, như : Github, Imgur, Instagram, Twitter...

<a name="1">
	
## I. Một số thuật ngữ

<a name="2">
	
### 1. Access Control List (ACL)

**ACLs** được dùng để test 1 vài điều kiện và thực hiện hành động (chọn server, block 1 request...) dựa vào kết quả kiểm tra. Sử dụng ACLs cho phép forward linh hoạt các network traffic dựa vào các nhân tố như pattern-matching và một số kết nối tới backend.

Ví dụ: `acl url_blog path_beg /blog`

ACL sẽ match nếu như phần request của user bắt đầu với /blog, nó sẽ match với request của `http://yourdomain.com/blog/blog-entry-1`

<a name="3">
	
### 2. Backend

Backend là tập hợp của các server mà nhận được request. Các backed được định nghĩa trong section "backend" của file cấu hình HAProxy. Trong phần lớn form cơ bản, 1 backend được định nghĩa bởi thuật toán cân bằng tải nào được dùng và danh sách các server và port.

Một backend có thể có một hoặc nhiều server trong nó, thêm nhiều server vào backend sẽ giúp việc cân bằng tải linh hoạt hơn

Thuật toán sử dụng là `roundrobin`, `mode http` chỉ ra rằng layer 7 proxying đuợc dùng. option `check` chỉ ra rằng các check sẽ được thực hiện ở các server này.

VD : Cấu hình 2 backend, web-backend và blog-backend với 2 web server :
```
backend web-backend
	   balance roundrobin
	   server web1 web1.yourdomain.com:80 check
	   server web2 web2.yourdomain.com:80 check

backend blog-backend
	   balance roundrobin
	   mode http
	   server blog1 blog1.yourdomain.com:80 check
	   server blog1 blog1.yourdomain.com:80 check
```

<a name="4">
	
### 3. Frontend

**Frontend** định nghĩa việc các request nên được chỏ tới cách backend như thế nào. **Frontend** được định nghĩa trong section **frontend** của file cấu hình HAProxy :

- Set các IP và port (vd 10.1.1.7:80, * :443...).
- ACLs.
- `use_backend` rules, định nghĩa ra các backend nào sẽ được dùng dựa vào điều kiện ACL được match, and/or một "default_backend" rule kiểm soát các case khác.
- Tóm lại nó dùng để khai báo danh sách các sockets đang lắng nghe kết nối để cho phép client kết nối tới.

Một frontend có thể được cấu hình theo nhiều kiểu của network traffic.

<a name="5">
	
## II. Các dạng của Load Balancing

<a name="6">
	
### 1. Layer 4 Load Balancing

Cân bằng tải kiểu này sẽ chỏ user traffic dựa vào IP range và port (VD : 1 request tới từ http://yourdomain.com/anything, traffic sẽ được chỏ tới backend mà được kiểm soát bởi yourdomain.com trên port 80)

<img src="https://i.imgur.com/xuXhdkH.png">

User truy cập tới load balancer, request của user sẽ được chỏ tới `web-backend` group của các backend server. Bất kỳ backend server nào được chọn sẽ phản hồi trực tiếp tới request của user. Thông thường, tất cả các user trong `web-backend` nên có nội dung thống nhất, trong khi user nhận được nội dung không đồng nhất. Chú ý rằng các server kết nối tới cùng một database

<a name="7">
	
### 2. Layer 7 Load Balancing

<img src="https://i.imgur.com/9FKaudn.png">

Nếu người dùng request yourdomain.com/blog, chúng được chỏ tới `blog` backend - một set các server chạy 1 khối ứng dụng. Các request khác được chỏ tới `web-backend`, chạy ứng dụng khác. Cả 2 backend đều được chỏ tới cùng 1 database.. 
VD :
```
frontend http 
	bind *:80 
	mode http
	
	acl url_blog path_beg /blog
	use_backend blog-backend if url_blog
	
	default_backend web-backend
```
**Giải thích**:

- `mode` là "http", kiểm soát tất cả incoming traffic trên port 80.
- `acl url_blog path_beg /blog`: match với một request nếu như phần đầu của user bắt đầu với /blog.
- `use_backend blog-backend if url_blog`: sử dụng ACLs để proxy traffic tới blog-backend.
- `default_backend web-backend`: chỉ ra tất cả các traffic khác sẽ chỏ tới web-backend.

<a name="8">
	
### 3. Các thuật toán Cân bằng tải

Các thuật toán cân bằng tải được dùng để chỉ ra rằng server nào trong 1 backend sẽ được chọn khi cân bằng tải. HAProxy đưa ra 1 vài lựa chọn cho các thuật toán, các server có thể được gán parameter `weight` để đánh giá server có được chọn thường xuyên không, so sánh với các server khác.

Một vài thuật toán thông dụng:

- **Round Robin** chọn server nào được quay. Đây là thuật toán mặc định.

- **Leastconn** chọn server với số các kết nối ít nhất - được đề xuất với các session dài hạn. Các server trong cùng 1 backend được quay vòng với **round-robin**

- **Source** chọn server dể dùng dựa vào source IP. Phương thức này đảm bảo user sẽ kết nối tới cùng 1 server. Dùng cho các máy chủ chạy SSL, máy chủ phụ thuộc vào địa chỉ nguồn của client , ví dụ IP address của người dùng của bạn.

Ngoài ra còn 1 số giải thuật như **uri**, **hdr**, **first**

<a name="9">
	
### 4. Sticky Session

Một vài ứng dụng yêu cầu user tiếp tục kết nối tới cùng backend server. Điều này được thực hiện bởi sticky session, yêu cầu sử dụng appsession parameter ở trên backend.

<a name="10">
	
### 5. Health Check

HAProxy sử dụng health checks để xác định nếu 1 backend server khả dụng cho quá trình request, nó sẽ tránh được việc nếu backend không còn khả dụng, thì việc remove server ra khỏi backend đó phải làm bằng tay. Mặc định thì health check sẽ cố gắng khởi tạo 1 kết nối TCP tới server, nó kiểm tra nếu backend server được lắng nghe trên IP và port được cấu hình.

Nếu 1 server thất bại trong việc health check, vậy nên không thể tiếp nhận request, nó sẽ tự động tắt trong backend, traffic sẽ chỉ chỏ đến nó đến khi nó khả dụng trở lại. Nếu tất cả các server trong backend fail, service sẽ không khả dụng đến khi có ít nhất 1 trong các backend server khả dụng trở lại.

<a name="11">
	
## III. Mô hình kết hợp với keepalive


Đối với các cài đặt load balancing layer 4/7 phía trên , chúng sử dụng 1 load balancer để điều khiển traffic tới một hoặc nhiều backend server. tuy nhiên nếu load balancer bị lỗi thì dữ liệu sẽ bị ứ đọng dẫn tới downtime (bottleneck - nghẽn cổ chai). keepalived sinh ra để giải quyết vấn đề này.

<img src="https://i.imgur.com/nBZWycp.gif">

Ở ví dụ trên, ta có nhiều load balancer (1 active và một hoặc nhiều passive). Khi người dùng kết nối đến một server thông qua ip public của active load balancer, nếu load balancer ấy fails, phương thức failover sẽ detect nó và tự động gán ip tới 1 passive server khác.


### So sánh HAProxy và một số giải pháp LB khác

**Một số giải pháp LB khác**

- Linux Virtual Server (LVS) – Một layer 4 load balancer đơn giản, nhanh được giới thiệu trong nhiều bản phân phối Linux.

- Nginx – Máy chủ web nhanh và đáng tin cậy được dùng cho proxy và cân bằng tải. Nginx thường dùng kết hợp với HAProxy cho việc lưu đệm (caching) và nén dữ liệu (compression) của mình.
