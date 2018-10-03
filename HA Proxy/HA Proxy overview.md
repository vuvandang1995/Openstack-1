# Tìm hiểu về HA Proxy

**HAProxy - High Availability Proxy**: Mục đích sử dụng là cân bằng tải TCP/HTTP

- Phần mềm mã nguồn mở Load Balancer và proxying TCP/HTTP chạy trên Linux, Solaris và FreeBSD

- Được dùng nhằm tăng hiệu năng và đáp ứng sự phân tán khối lượng công việc trên các server (web, ứng dụng, database...)

- Được sử dụng trong rất nhiều môi trường cấu hình cao, như : Github, Imgur, Instagram, Twitter...


## I. Một số thuật ngữ

### 1. Access Control List (ACL)

**ACLs** được dùng để test 1 vài điều kiện và thực hiện hành động (chọn server, block 1 request...) dựa vào kết quả kiểm tra. Sử dụng ACLs cho phép forward linh hoạt các network traffic dựa vào các nhân tố như pattern-matching và một số kết nối tới backend.

Ví dụ: `acl url_blog path_beg /blog`

ACL sẽ match nếu như phần request của user bắt đầu với /blog, nó sẽ match với request của http://yoursite.com/blog/blog-entry

### 2.Backend

Backend là tập hợp của các server mà nhận được request. Các backed được định nghĩa trong section "backend" của file cấu hình HAProxy. Trong phần lớn form cơ bản, 1 backend được định nghĩa bởi thuật toán cân bằng tải nào được dùng và danh sách các server và port.

Một backend có thể có một hoặc nhiều server trong nó, thêm nhiều server vào backend sẽ giúp việc cân bằng tải linh hoạt hơn

Thuật toán sử dụng là `roundrobin`, `mode http` chỉ ra rằng layer 7 proxying đuợc dùng. option `check` chỉ ra rằng các check sẽ được thực hiện ở các server này.

### 3. Frontend

**Frontend** định nghĩa việc các request nên được chỏ tới cách backend như thế nào. **Frontend** được định nghĩa trong section **frontend** của file cấu hình HAProxy :

- Set các IP và port (vd 10.1.1.7:80, * :443...).
- ACLs.
- `use_backend` rules, định nghĩa ra các backend nào sẽ được dùng dựa vào điều kiện ACL được match, and/or một "default_backend" rule kiểm soát các case khác.
- Tóm lại nó dùng để khai báo danh sách các sockets đang lắng nghe kết nối để cho phép client kết nối tới.

Một frontend có thể được cấu hình theo nhiều kiểu của network traffic.

## II. Các dạng của Load Balancing

### 1. Layer 4 Load Balancing

Cân bằng tải kiểu này sẽ chỏ user traffic dựa vào IP range và port (VD : 1 request tới từ http://yourdomain.com/anything, traffic sẽ được chỏ tới backend mà được kiểm soát bởi yourdomain.com trên port 80)

<img src="https://i.imgur.com/xuXhdkH.png">

User truy cập tới load balancer, request của user sẽ được chỏ tới `web-backend` group của các backend server. Bất kỳ backend server nào được chọn sẽ phản hồi trực tiếp tới request của user. Thông thường, tất cả các user trong `web-backend` nên có nội dung thống nhất, trong khi user nhận được nội dung không đồng nhất. Chú ý rằng các server kết nối tới cùng một database

### 2. Layer 7 Load Balancing
  

