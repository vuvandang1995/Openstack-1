# Phần này sẽ tìm hiểu file cấu hình của nova

-	Xác thực keystone
```
[DEFAULT]
auth_strategy = keystone
```

-	Bật compute API và metadata API
```
[DEFAULT]
enabled_apis = osapi_compute,metadata
```

-	Truy cập hàng đợi message rabbitmq
```
[DEFAULT]
transport_url = rabbit://openstack:ducnm37@192.168.40.61
```

-	Thiết lập ip quản lý controller
```
[DEFAULT]
my_ip = 192.168.40.61 
```

-	Bật tính năng hỗ trợ network
```
[DEFAULT]
use_neutron = True 
firewall_driver = nova.virt.firewall.NoopFirewallDriver 
```


-	Kết nối tới database
```
[api_database]
connection = mysql+pymysql://nova:ducnm37@192.168.40.61/nova_api
[database]
connection = mysql+pymysql://nova:ducnm37@192.168.40.61/nova
```

-	Cấu hình API kết nối tới dịch vụ image
```
[glance]
api_servers = http://192.168.40.61:9292
```

-	Truy cập xác thực dịch vụ
```
[api]
auth_strategy = keystone
```

-	Cấu hình xác thực nova qua keystone
```
[keystone_authtoken]
auth_url = http://192.168.40.61:5000/v3 ( admin Identity API endpoint)
memcached_servers = 192.168.40.61:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = ducnm37
```

-	Cấu hình truy cập neutron
```
[neutron]
url = http://192.168.40.61:9696
auth_url = http://192.168.40.61:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = ducnm37
service_metadata_proxy = true
metadata_proxy_shared_secret = ducnm37
```

-	Cấu hình lock path
```
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```

-	Cấu hình Placement API
```
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.40.61:5000/v3
username = placement
password = ducnm37
```

-	Cấu hình vnc proxy xử dụng ip quản lý controller
```
[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
```

