# các tham số trong option global

### Process management and security:

- ca-base <dir> : Gán thư mục mặc định để tìm nạp chứng chỉ CA SSL và CRLs khi được sử dụng với đường dẫn `ca-file` hoặc `crl-file` 

- chroot <jail dir> : Thay đổi quyền root, tăng mức độ bảo mật khi hệ thống bị xâm nhập, và quyền thao tác hệ thống chỉ có superuser 

- cpu-map [auto:]<process-set>[/<thread-set>] <cpu-set>... : Gán tiến trình với ràng buộc về luồng cpu
```
cpu-map 1-4 0-3   # bind processes 1 to 4 on the first 4 CPUs

cpu-map 1/all 0-3 # bind all threads of the first process on the
                  # first 4 CPUs

cpu-map 1- 0-     # will be replaced by "cpu-map 1-64 0-63"
                  # or "cpu-map 1-32 0-31" depending on the machine's
                  # word size.
```

- crt-base <dir> :Gán thư mục mặc định để tìm nạp chứng chỉ SSL khi sử dụng crtfile
  
- daemon : Thực hiện các tiến trình trong backgroud

- deviceatlas-json-file <path> : Tải file json bởi API

- deviceatlas-log-level <value> : Đặt mức thông tin được API trả lại. Chỉ thị này là tùy chọn và đặt thành 0 theo mặc định nếu không được đặt

- deviceatlas-separator <char> : Đặt phân cách cho kết quả trả về của API
  
- deviceatlas-properties-cookie <name> : Đặt tên cookie sử dụng 
  
- external-check : Check các rules thêm vào

- gid <number> : Thay đổi group ID thành 1 số nó là group ID giành riêng cho HAProxy
  
- hard-stop-after <time> : Xác định thời gian tối đã thực hiện soft-stop, tính bằng ms
```
global
hard-stop-after 30s
```
  
- group <group name> : Group id
  
- log <address> [len <length>] [format <format>] <facility> [max level [min level]]  : Thiết lập log

```
Address: địa chỉ ip cần lấy log
  
Length: có giá trị từ 80 to 65535, giá trị mặc định là 1024 dòng

Format: có 2 dạng là rfc3164 và rfc5424

Facility: là 1 trong các chuẩn kern user mail daemon auth syslog lpr newsuucp cron auth2 ftp ntp audit alert cron2 local0 local1 local2 local3 local4 local5 local6 local7

max level, min level: emerg alert crit err warning notice info debug
```

- log-send-hostname [<string>] : set trường hostname vào trong syslog header


