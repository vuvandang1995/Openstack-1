# QOS

## 1. QoS là gì và để làm gì?

QoS được hiểu là khả năng đảm bảo về các yêu cầu liên quan tới network như bandwidth, latency, jitter và tính tin cậy trong việc tuân thủ theo Service Level Agreement (SLA) giữa nhà cung cấp ứng dụng và người dùng.

Ví dụ đơn giản đó là chúng ta có thể set bandwidth cho từng loại traffic (những traffic như Voice over IP, streaming,... cần được ưu tiên hơn chẳng hạn)

QoS policies có thể được áp dụng:

- Theo từng network: Tất cả các ports được gán vào network nơi có QoS policies sẽ được áp dụng.
- Theo từng port: Các port cụ thể sẽ được áp dụng các policies, khi port đã có policies rồi thì nó sẽ bị ghi đè.

Trong OPS, QoS là một advanced service plug-in, nó được khai báo trong code của neutron và cung cấp thông qua ml2 extension driver.

<a name="2"></a>
## 2. Hướng dẫn cấu hình QoS network trong OPS

- Cấu hình thêm qos service trong file `/etc/neutron/neutron.conf` ở node network

``` sh
[DEFAULT]

service_plugins = router,qos
```

- Khai báo trong file `/etc/neutron/plugins/ml2/ml2_conf.ini`

``` sh
[ml2]
extension_drivers = port_security, qos
```

- Khai báo trong file `/etc/neutron/plugins/ml2/openvswitch_agent.ini`

``` sh
[agent]
extensions = qos
```

- Trên compute node, khai báo extensions trong file `/etc/neutron/plugins/ml2/openvswitch_agent.ini`

``` sh
[agent]
extensions = qos
```

**Hướng dẫn tạo QoS policy and its bandwidth limit rule:**

Mặc định thì chỉ có admins mới có thể tạo QoS policies , bạn có thể thay đổi cấu hình trong file `policy.json` để cho phép người dùng trên các projects xem được và tạo QoS policies.

Ví dụ:

``` sh
"get_policy": "rule:regular_user",
"create_policy": "rule:regular_user",
"update_policy": "rule:regular_user",
"delete_policy": "rule:regular_user",
```

- Câu lệnh để tạo policy mới:

``` sh
neutron qos-policy-create bw-limiter

[root@controller ~(keystone_admin)]# neutron qos-policy-create bw-limiter
Created a new policy:
+-----------------+--------------------------------------+
| Field           | Value                                |
+-----------------+--------------------------------------+
| created_at      | 2017-10-11T02:28:50Z                 |
| description     |                                      |
| id              | 349ba7d1-95f7-437b-aa82-24a907550ce2 |
| name            | bw-limiter                           |
| project_id      | fe365fdcc17c487f9cf7d375fa01a379     |
| revision_number | 1                                    |
| rules           |                                      |
| shared          | False                                |
| tenant_id       | fe365fdcc17c487f9cf7d375fa01a379     |
| updated_at      | 2017-10-11T02:28:50Z                 |
+-----------------+--------------------------------------+
```

- Set up bandwidth cho policy

``` sh
neutron qos-bandwidth-limit-rule-create --max-kbps 102400 --max-burst-kbps 1000 bw-limiter
```

**Lưu ý:**

Mặc định traffic sẽ được limit theo chiều egress đối với các port được áp policy, để thay đổi sang chiều ingress, sử dụng câu lệnh sau:

``` sh
openstack network qos rule create --type bandwidth-limit --max-kbps 3000 \
    --max-burst-kbits 300 --ingress bw-limiter
```

Tuy nhiên câu lệnh này chỉ có tác dụng trong một vài bản OPS gần đây, những bản cũ không thể áp policy theo chiều ingress được

- Để hiển thị các port, sử dụng câu lệnh sau:

``` sh
neutron port-list
```

- Áp rule vào port cần thiết lập

`neutron port-update ID_PORT --qos-policy bw-limiter`

- Để áp policy vào network, sử dụng câu lệnh

``` sh
openstack network set --qos-policy bw-limiter private
```

Tuy nhiên câu lệnh này cũng không thể áp dụng với các bản OPS cũ như mitaka, newton

- Bỏ rule của port vừa thiết lập

`neutron port-update ID_PORT --no-qos-policy`

## 3. Lab cài đặt và sử dụng Nuttcp

**Cài đặt nuttcp**

Đối với CentOS

`yum install epel-release`

`yum install nuttcp`

Đối với Ubuntu

`apt-get install nuttcp`

**Một vài kịch bản test cơ bản**

Server

`nuttcp -S -p12345  (*dùng trên port 12345,nếu ko có p12345 mặc định nuttcp sử dụng port 5000)`

Client

`nuttcp -p12345 <serverhost>  (dùng trên port 12345,nếu ko có p12345 mặc định nuttcp sử dụng port 5000)` 

Câu lệnh này sử dụng phương thức test mặc định đó là gửi các gói tin tcp trong vòng 10 giây

``` sh
[root@thaonv ~]# nuttcp 172.16.69.239
 1986.7500 MB /  10.00 sec = 1666.2098 Mbps 59 %TX 55 %RX 0 retrans 1.13 msRTT
```

%TX và %RX là mức độ sử dụng CPU trên transmitter và receiver.


Server

`nuttcp -S`

Client

`nuttcp -w6m <serverhost>`

Gần giống câu lệnh trên nhưng window size sẽ được đẩy lên cao hơn


Server

`nuttcp -S`

Client

`nuttcp -u -i -Ri50m <serverhost>`

Thường sử dụng để test số lượng packet bị mất. Câu lệnh trên sẽ truyền trong 10 giây các gói tin udp với tốc độ 50 Mbps. Nó sẽ trả về 1 report mỗi giây.

``` sh
[root@thaonv ~]# nuttcp -u -i -Ri50m 172.16.69.239
    5.8740 MB /   1.00 sec =   49.2634 Mbps     0 /  6015 ~drop/pkt  0.00 ~%loss
    5.8672 MB /   1.00 sec =   49.2243 Mbps     0 /  6008 ~drop/pkt  0.00 ~%loss
    5.7988 MB /   1.00 sec =   48.6393 Mbps    40 /  5978 ~drop/pkt  0.67 ~%loss
    5.8281 MB /   1.00 sec =   48.8904 Mbps     8 /  5976 ~drop/pkt  0.13 ~%loss
    5.8330 MB /   1.00 sec =   48.9331 Mbps     0 /  5973 ~drop/pkt  0.00 ~%loss
    5.8613 MB /   1.00 sec =   49.1726 Mbps     0 /  6002 ~drop/pkt  0.00 ~%loss
    5.7812 MB /   1.00 sec =   48.4886 Mbps    79 /  5999 ~drop/pkt  1.32 ~%loss
    5.8408 MB /   1.00 sec =   49.0039 Mbps    12 /  5993 ~drop/pkt  0.20 ~%loss
    5.8398 MB /   1.00 sec =   48.9850 Mbps     0 /  5980 ~drop/pkt  0.00 ~%loss

   58.3477 MB /  10.00 sec =   48.9496 Mbps 99 %TX 9 %RX 139 / 59887 drop/pkt 0.23 %loss
```

Client

`nuttcp -w1m 127.0.0.1`

Dùng để test tốc độ bên trong của host.

``` sh
[root@thaonv ~]# nuttcp -w1m 127.0.0.1
21836.0000 MB /  10.00 sec = 18317.4542 Mbps 98 %TX 92 %RX 0 retrans 0.18 msRTT
```
