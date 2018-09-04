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
