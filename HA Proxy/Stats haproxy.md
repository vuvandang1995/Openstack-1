# Thống kê hoạt động của haproxy

## Mục lục

[1. Cấu hình Stats cho HAProxy](#1)

[2. Các thông số trên web browser](#2)

[3. Cách thu thập stats qua Unix Socket Interface](#3)

-----------------

<a name="1"></a>
## 1. Cấu hình Stats cho HAProxy

- Để cho phép HAProxy hiển thị số liệu thống kê, thêm một số dòng sau vào trong file config ở setcion default

```
stats enable
stats uri /monitor
stats auth ducnm37:123456
```

- Trong đó `stats uri` là đường dẫn tới trang hiển thị số liệu thống kê và `stats auth` là thông tin tài khoản và mật khẩu để tuy cập.

- Để theo dõi HAProxy Stats, tiến hành truy cập vào địa chỉ http://192.168.40.70/monitor với 192.168.40.70 là ip VIP, username là `ducnm37` và password là `123456`

<img src="https://i.imgur.com/ClhEkGI.png">

<a name="2"></a>
## 2. Các thông số trên web browser

Như ta thấy trong hình thì phần phía trên là thông tin chung về process của HAProxy, đây là những thông tin thường được set trong phần `global` của file config.

Một số thông tin đáng chú ý ở đây đó là `pid` (process id), `uptime` thời gian uptime kể từ lần restart cuối cùng, `current conns` số connection đang kết nối tới server.

- `ulimit-n` : số lượng file-descriptors tối đa cho mỗi process, nó thường được tính toán tự động
- `memmax` : số lượng RAM tối đa cho mỗi process, mặc định giá trị này là 0 túc là unlimited
- `maxsock` : số lượng sockets tối đa mà HAProxy process có thể tạo ra. giống với `ulimit-n`, giá trị này thường được tính toán tự động
- `maxconn` : Số lượng current connection tối đa cho mỗi process
- `maxpipes` : Số lượng pipes tối đa cho mỗi process, hiện tại thì trong haproxy, pipes chỉ được sử dụng cho kernel-based tcp splicing.
- `idle` : idle time là thời gian chờ. Nếu tỉ lệ này lên tới 100% thì có nghĩa rằng load đang rất thấp.

Bên cạnh đó là các phần giải thích cho trạng thái của các frontend và backend mà HAProxy đnag quản lí theo trạng thái up/down.

Phía dưới là phần thông số chi tiết cho các service mà haproxy đang quản lí. Sau đây là những phần giải thích những thông số ở các bảng này.

**Qeue**

- `Cur` : Những request hiện đang ở trong qeue
- `Max` : Giá trị tối đa của Cur

**Session**

- `Max` : Số Session tối đa
- `Limit` : Số Session limit được config
- `Total` : Số Session cộng dồn

**Bytes**

- `In` : Số bytes vào
- `Out` : Số byte ra

**Denied**

- `Req` : Các request bị denied bởi nó match với các rule được define.
- `Resp` :  Các responses bị denied bởi nó match với các rule hoặc `option checkcache`

**Errors**

- `Req` : Request lỗi bởi một số nguyên nhân sau: bị terminate bởi client trước khi gửi, read error từ phía client, client timeout, client đóng kết nối, nhiều bad requests từ client,...
- `Conn` : Số request bị error khi cố kết nối tới backend server
- `Resp` : Response error

**Warnings**

- `Retr` : Số lần connection tới server bị retry.
- `Redis` : Số lần request gửi đi lại sang server khác.

**Server**

- `Status`: trạng thái (UP/DOWN/NOLB/MAINT/MAINT(via)/MAINT(resolution)...)
- `LastChk` : Trạng thái của lần check health gần nhất
  - UNK     -> unknown
  - INI     -> initializing
  - SOCKERR -> socket error
  - L4OK    -> check passed on layer 4, no upper layers testi enabled
  - L4TOUT  -> layer 1-4 timeout
  - L4CON   -> layer 1-4 connection problem, for example
                   "Connection refused" (tcp rst) or "No route to host" (icmp)
  - L6OK    -> check passed on layer 6
  - L6TOUT  -> layer 6 (SSL) timeout
  - L6RSP   -> layer 6 invalid response - protocol error
  - L7OK    -> check passed on layer 7
  - L7OKC   -> check conditionally passed on layer 7, for examp 404 with
                   disable-on-404
  - L7TOUT  -> layer 7 (HTTP/SMTP) timeout
  - L7RSP   -> layer 7 invalid response - protocol error
  - L7STS   -> layer 7 response error, for example HTTP 5xx

- `Wght`: Tổng số weight (backend), server weight (server)
- `Act`: Số server (backend) đang active
- `Bck`: Số server backup
- `Chk`: Số lần check failed
- `Dwn`: Số lần UP -> Down
- `Dwntime`: Tổng thời gian downtime (theo giây)
- `Thrtle`: Số phần trăm Throttle cho server

<a name="3"></a>
## 3. Cách thu thập stats qua Unix Socket Interface

Ngoài Statistic trên http, HAProxy cũng cho phép bạn thu thập metrics qua unix socket. Để cấu hình, thêm dòng sau tại phần **global** trong file cấu hình haproxy

```
stats socket /var/run/haproxy.sock mode 600 level admin
stats timeout 2m #Wait up to 2 minutes for input
```

Dùng netcat tool để giao tiếp với socket. Với câu lệnh sau, bạn sẽ nhận được:

- Thông tin về process HAProxy đang chạy
- Thông số của frontend và backup

`echo "show info;show stat" | nc -U /var/run/haproxy.sock`

<img src="https://i.imgur.com/fl8aYU7.png">


Nếu muốn vào Interactive mode, sử dụng câu lệnh sau

`nc -U /var/run/haproxy.sock`

Sau đó vào prompt bằng cách gõ vào `prompt`

`quit` để thoát.
