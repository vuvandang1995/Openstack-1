# các tham số trong cấu hình HAproxy

## Một số tham số thường sử dụng trong Global

- `daemon` : Thực hiện các tiến trình trong backgroud

- `group <group name>` : Group dùng cho HAproxy
  
- `log <address> [len <length>] [format <format>] <facility> [max level [min level]]` : Thiết lập log

```
Address: địa chỉ ip để lưu log, hoặc 1 đường dẫn để lưu log.
  
Length: có giá trị từ 80 to 65535, giá trị mặc định là 1024 dòng

Format: có 2 dạng là rfc3164 và rfc5424

Facility: là 1 trong các chuẩn kern user mail daemon auth syslog lpr newsuucp cron auth2 ftp ntp audit alert cron2 local0 local1 local2 local3 local4 local5 local6 local7

max level, min level: emerg alert crit err warning notice info debug
```

- `maxconn <number>` : Đặt số kết nối tối đa cho mỗi phiên 
  
- `pidfile <pidfile>` : Ghi PIDs của tất cả daemons vào trong file pidfile

- `tune.bufsize <number>` : Đặt kích thước bộ đệm buffer (tính theo bytes), giá trị thấp hơn cho phép nhiều hơn các phiên cùng tồn tại trong cùng một lượng RAM và các giá trị cao hơn cho phép một số các ứng dụng có cookie rất lớn để hoạt động, nếu tăng buffer thì ta cần giảm maxconn xuống để tránh trường hợp hết bộ nhớ. Giá trị mặc định là 16384 và có thể được thay đổi. Ngoài ra, sử dụng các yêu cầu HTTP  giá trị này phải là 16384 trở lên. Nếu một Yêu cầu HTTP lớn hơn (tune.bufsize - tune.maxrewrite) haproxy sẽ trả lại lỗi HTTP 400 (Yêu cầu Không hợp lệ), tương tự nếu một phản hồi HTTP lớn hơn so với kích thước này, haproxy sẽ trả về HTTP 502 (Bad Gateway).

- `tune.maxrewrite <number>` : Kích thước giành riêng (tính bằng bytes) trong buffersize để viết lại các header hoặc các trường khác, mặc định nó chiếm 1 nửa kích thước của bộ đệm buffer, giá trị khuyến cáo là 1024


- `user <username> [password|insecure-password <password>] [groups <group>,<group>,(...)]`  : Khai báo user sử dụng cho HAproxy

```
userlist L1
  group G1 users tiger,scott
  group G2 users xdb,scott

  user tiger password $6$k6y3o.eP$JlKBx9za9667qe4(...)xHSwRv6J.C0/D7cV91
  user scott insecure-password elgato
  user xdb insecure-password hello

userlist L2
  group G1
  group G2

  user tiger password $6$k6y3o.eP$JlKBx(...)xHSwRv6J.C0/D7cV91 groups G1
  user scott insecure-password elgato groups G1,G2
  user xdb insecure-password hello groups G2
```

- `stats socket [<address:port>|<path>] [param*]` : Liên kết một UNIX socket tới <path> , hoặc 1 TCPv4/v6 address tới <address:port>
