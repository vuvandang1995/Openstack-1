# File cấu hình keepalive

### 1. Cấu trúc

<img src="https://i.imgur.com/jIhBFdR.png">


Gồm 2 thành phần chính là Health-checker và VRRP 

- Health-checker worker threads: mỗi một health-check được đăng kí với global scheduling framework. Nó sẽ đảm đương nhiệm vụ này bằng cách sử dụng Keepalived health-check framework. health-check framework chứa 3 checkers:

  - TCP_CHECK: Hoạt động ở layer 4. Nếu remote server không trả lời thì server sẽ bị loại khỏi server pool.
  
  - HTTP_GET: Hoạt động ở layer 5. Thực hiện GET HTTP với các url. Kết quả sẽ được tổng hợp lại sử dụng MD5 algorithm nếu nó ko match với giá trị mong muốn, server sẽ bị loại khỏi server pool. Module này hoạt động hiệu quả khi server bạn chạy nhiều hơn một ứng dụng.
  
  - SSL_GET: giống với HTTP_GET nhưng sử dụng ssl connection để kết nối tới server.
  
  - MISC_CHECK: cho phép người dùng tự tạo script để chạy như là Health-checker. Kết quả buộc phải là 0 hoặc 1.

- VRRP Packet Dispatcher: Phân tách I/O để quản lí các VRRP Instance

Hai thành phần này sử dụng những công cụ sau:

  - SMTP notification: cho phép keepalived gửi cảnh báo
  
  - IPVS framework: LVS kernel interface cho việc quản lí real server pool
  
  - Netlink: Kernel routing interface cho VRRP. Quản lí VRRP VIP
  
  - Multicast: Dùng để gửi VRRP adverts
  
  - NETFILTER
  
  - SYSLOG
  
### 2. Cấu hình keepalived

#### Global definitions

``` sh
global_defs {
  notification_email {
    email
    email
  }
  notification_email_from email
  smtp_server host
  smtp_connect_timeout num
  lvs_id string
}
```

- `global_defs` : cho biết đây là block cấu hình của global def
- `notification_email` : email nhận được thông báo
- `notification_email_from` : email được sử dụng khi xử lí câu lệnh “MAIL FROM:” SMTP
- `smtp_server` : SMTP server dùng để gửi mail thông báo
- `smtp_connection_timeout` : timeout cho tiến trình xử lí SMTP

<a name="4.2"></a>
#### Virtual server definitions

``` sh
virtual_server (@IP PORT)|(fwmark num) {
    delay_loop num
    lb_algo rr|wrr|lc|wlc|sh|dh|lblc
    lb_kind NAT|DR|TUN
    (nat_mask @IP)
    persistence_timeout num
    persistence_granularity @IP
    virtualhost string
    protocol TCP|UDP

    sorry_server @IP PORT

    real_server @IP PORT {
      weight num
      TCP_CHECK {
        connect_port num
        connect_timeout num
      }
    }
    real_server @IP PORT {
      weight num
      MISC_CHECK {
        misc_path /path_to_script/script.sh
        (or misc_path “/path_to_script/script.sh <arg_list>”)
      }
    }
    real_server @IP PORT {
        weight num
        HTTP_GET|SSL_GET {
            url { # You can add multiple url block
              path alphanum
              digest alphanum
            }
            connect_port num
            connect_timeout num
            nb_get_retry num
            delay_before_retry num
        }
    }
}
```

- `delay_loop` : số giây giữa các lần check
- `lb_algo` : load balancing algorithm (rr|wrr|lc|wlc…)
- `lb_kind` : method dùng để forwarding (NAT|DR|TUN)
- `persistence_timeout` : thời gian timeout cho các persistence connection
- `Virtualhost` : HTTP virtualhost để dùng cho  HTTP|SSL_GET
- `protocol` :  (TCP|UDP)
- `sorry_server` : server được add vào pool nếu mọi server đều bị down
- `Weight` : trọng số cho load balancing
- `TCP_CHECK` : check bằng tcp connect
- `MISC_CHECK` : check bằng user defined script
- `misc_path` : Đường dẫn tới script
- `HTTP_GET` : check bằng HTTP GET request
- `SSL_GET` : check bằng SSL GET request

<a name="4.3"></a>
#### VRRP Instance definitions

``` sh
vrrp_sync_group string {
  group {
    string
    string
  }
  notify_master /path_to_script/script_master.sh
      (or notify_master “/path_to_script/script_master.sh <arg_list>”)
  notify_backup /path_to_script/script_backup.sh
      (or notify_backup “/path_to_script/script_backup.sh <arg_list>”)
  notify_fault /path_to_script/script_fault.sh
      (or notify_fault “/path_to_script/script_fault.sh <arg_list>”)
}
vrrp_instance string {
  state MASTER|BACKUP
  interface string
  mcast_src_ip @IP
  lvs_sync_daemon_interface string
  virtual_router_id num
  priority num
  advert_int num
  smtp_alert
  authentication {
    auth_type PASS|AH
    auth_pass string
  }
  virtual_ipaddress { # Block limited to 20 IP addresses
    @IP
    @IP
    @IP
  }
  virtual_ipaddress_excluded { # Unlimited IP addresses number
    @IP
    @IP
    @IP
  }
  notify_master /path_to_script/script_master.sh
    (or notify_master “/path_to_script/script_master.sh <arg_list>”)
  notify_backup /path_to_script/script_backup.sh
    (or notify_backup “/path_to_script/script_backup.sh <arg_list>”)
  notify_fault /path_to_script/script_fault.sh
    (or notify_fault “/path_to_script/script_fault.sh <arg_list>”)
}
```

- `State` : trạng thái của Instance
- `Interface` : Interface mà Instance đang chạy
- `mcast_src_ip` : địa chỉ Multicast
- `lvs_sync_daemon_inteface` : Interface cho LVS sync_daemon
- `Virtual_router_id` :  VRRP router id
- `Priority` : thứ tự ưu tiên trong VRRP router
- `advert_int` : số  advertisement interval trong 1 giây
- `smtp_aler` : kích hoạt thông báo SMTP cho MASTER
- `authentication` :  VRRP authentication
- `virtual_ipaddress` : VRRP VIP
