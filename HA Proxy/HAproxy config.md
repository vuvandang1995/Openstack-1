# cấu hình file HAproxy

### 1. Cách thức hoạt động HAproxy

HAProxy là single-threaded, event-driven, non-blocking engine kết hợp các I/O layer với priority-based scheduler. Vì nó được thiết kế với mục tiêu vận chuyển dữ liệu, kiến trúc của nó được tối ưu hóa để chuyển dữ liệu nhanh nhất có thể. Nó có những layer model với những cơ chế riêng để đảm bảo dữ liệu không đi tới những level cao hơn nếu không cần thiết. Phần lớn những quá trình xử lí diễn ra ở kernel và HAProxy làm mọi thứ tốt nhất để giúp kernel làm việc nhanh nhất có thể.

HAProxy (product) chỉ yêu cầu haproxy (programs) được thực thi và file cấu hình để chạy. File cấu hình sẽ được đọc trước khi nó khởi động, sau đó HAProxy sẽ cố gắng gộp tất cả listening sockets và sẽ từ chối khởi động nếu có lỗi.

Một khi HAProxy được bật lên, nó thực hiện 3 điều:

- Xử lí các incoming connections
- Định kì check các trạng thái của server (health checks)
- Trao đổi thông tin với các haproxy nodes khác

Quá trình xử lí các incoming connections là phần phức tạp nhất và nó phụ thuộc vào rất nhiều cấu hình. Có thể tóm gọn nó lại trong 9 bước:

- Cho phép các connections từ listening sockets thuộc về các cấu hình trong phần frontend
- Áp dụng các frontend-specific processing rules đối với những connections này
- Chuyển các incoming connections tới "backend", nơi chứa các server và cả những kế hoạch load balancing cho nó
- Áp dụng những backend-specific processing rules cho các connections
- Dựa vào kế hoạch load balancing mà quyết định server sẽ nhận kết nối
- Áp dụng backend-specific processing rules cho dữ liệu được trả về từ server
- Áp dụng frontend-specific processing rules cho dữ liệu được trả về từ server
- Tạo log để ghi lại những gì đã xảy ra
- Đối với http, lặp lại bước 2 để đợi một request mới, nếu không có, tiến hành đóng kết nối.

### 2. Định dạng file cấu hình

Quá trình cấu hình cho HAProxy bao gồm 3 nguồn chính:

- Các câu lệnh từ command-line được ưu tiên trước
- "global" sections, nơi chứa process-wide parameters
- proxies sections, có thể lấy từ "defaults", "listen", "frontend" và "backend"


Các cấu hình proxy có thể được đặt trong 4 secions:

- defaults <name>
- frontend <name>
- backend  <name>
- listen   <name>

Trong đó:

- "defaults" chứa những parameters mặc định cho tất cả những sections sử dụng phần khai báo của nó
- "frontend" chứa danh sách listening sockets cho phép kết nối từ clients
- "backend" sections chứa danh sách các servers mà proxy sẽ kết nối và forward packets.
- "listen" định nghĩa proxy hoàn chỉnh, là sự kết hợp của frontend và backend. Nó thường chỉ được dùng cho TCP-only traffic

Hiện tại thì có 2 proxy mode được hỗ trợ đó là "tcp" và "http". Nếu sử dụng "tcp" thì HAProxy đơn giản chỉ forward các traffic giữa 2 sides. Nếu sử dụng "http" mode thì nó sẽ phần tích giao thức và có thể tương tác với chúng bằng cách chặn, chuyển hướng, thêm, sửa, xóa nội dung trong request hoặc responses.


### 3. Biến môi trường

Bến trong HAProxy được bao bọc bởi dấu nháy kép. Nó phải được bắt đầu bằng "$" và nằm bên trong dấu ngoặc nhọn ({}). Nó có thể chứa chữ cái hoặc dấu gạch dưới ( _ ) nhưng không được phép bắt đầu bằng chữ số.

Ví dụ:

``` sh
        bind "fd@${FD_APP1}"

        log "${LOCAL_SYSLOG}:514" local0 notice   # send to local server

        user "$HAPROXY_USER"
```

<a name="3.3"></a>
### 4. Time format

Thông thường các dịnh dạng thời gian trong HAProxy thường được biểu diễn theo định dạng milliseconds, tuy nhiên HAProxy cũng hỗ trợ nhiều định dạng khác:

  - us : microseconds. 1 microsecond = 1/1000000 second
  - ms : milliseconds. 1 millisecond = 1/1000 second. (Mặc định)
  - s  : seconds. 1s = 1000ms
  - m  : minutes. 1m = 60s = 60000ms
  - h  : hours.   1h = 60m = 3600s = 3600000ms
  - d  : days.    1d = 24h = 1440m = 86400s = 86400000ms

<a name="3.4"></a>
### 5. Ví dụ

``` sh
   # Simple configuration for an HTTP proxy listening on port 80 on all
   # interfaces and forwarding requests to a single backend "servers" with a
   # single server "server1" listening on 127.0.0.1:8000
   global
       daemon
       maxconn 256

   defaults
       mode http
       timeout connect 5000ms
       timeout client 50000ms
       timeout server 50000ms

   frontend http-in
       bind *:80
       default_backend servers

   backend servers
       server server1 127.0.0.1:8000 maxconn 32


   # The same configuration defined with a single listen block. Shorter but
   # less expressive, especially in HTTP mode.
   global
       daemon
       maxconn 256

   defaults
       mode http
       timeout connect 5000ms
       timeout client 50000ms
       timeout server 50000ms

   listen http-in
       bind *:80
       server server1 127.0.0.1:8000 maxconn 32
```

Test cấu hình bằng câu lệnh sau:

`$ sudo haproxy -f configuration.conf -c`

### 6 Hướng dẫn cấu hình để HAProxy đẩy log ra syslog

Vì HAProxy không cho phép nó access tới file system nên cách duy nhất đó là gửi logs thông qua UDP server (mặc định ở port 514).

Để có thể làm được điều này, ta cần khai báo dòng sau vào cấu hình trong section `global`

``` sh
log /dev/log    local0
log /dev/log    local1 notice
```

Sau đó thêm dòng sau vào "defaults" section.

`log global`

Chính sửa lại file cấu hình của rsyslogd `/etc/rsyslog.conf`

``` sh
$ModLoad imudp
$UDPServerAddress *
$UDPServerRun 514
```

Cuối cùng là restart lại haproxy. Test lại bằng cách chạy câu lệnh sau

`strace -tt -s100 -etrace=sendmsg -p <haproxy's pid>`

<a name="4.3"></a>
### 7 Tóm tắt và giải thích các lệnh trong HAproxy

Cấu trúc câu lệnh

`$ haproxy [<options>]*`

Trong đó `[<options>]*` là các tùy chọn đi kèm, nếu không có options nào được khai báo thì nó sẽ hiển thị help page.

Danh sách các options:

- -- <cfgfile>* : tất cả các câu lệnh phía sau kí tự `--` đều là đường dẫn tới thư mục hoặc file cần được load và process theo thứ tự được khai báo. nó thường được dùng khi phải load nhiều file cùng lúc. Gần giống với `-f`, khác ở chỗ `-f` phải xuất hiện ở mỗi một file được khai báo đơn lẻ.
- -f <cfgfile|cfgdir> : Thêm <cfgfile> vào danh sách các file cấu hình được load. Mỗi một file chứa cấu hình của một hoặc nhiều section vì thế nó cần được bắt đầu bằng ` "global", "defaults", "peers", "listen", "frontend", "backend"`
- -C <dir> : Thay đổi thư mục trước khi load files cấu hình
- -D : Khởi động như một daemon. Lỗi sẽ không được thông báo trên terminal.
- -Ds: chạy ở mode systemd
- -L <name> : thay đổi local peer name thành <name>
- -N <limit> : Thiết lập maxconn trên mỗi proxy (mặc định là 2000)
- -V : bật verbose mode
- -c : Check file cấu hình, nếu mọi thứ ok thì output của nó sẽ không hiện gì
- -d : bật debug mode, mặc định sẽ tắt daemon mode
- -dG : tắt hàm getaddrinfo() để chuyển host names thành địa chỉ.
- -m <limit> : giới hạn số memory theo megabytes trên từng tiến trình
- -n <limit> : giới hạn các connection trên từng process. giống với <maxconn> trong global section.
- -p <file> : ghi tất cả các processes' pids vào một file trong quá trình khởi động, giống với `pidfile` trong section global
- -q: đặt mode quiet
- -v : thông báo version, ngày cài đặt
