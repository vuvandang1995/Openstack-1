# File cấu hình config neutron

- Cấu hình metadata: khai báo host và mật khẩu để lấy API cung cấp metadata
```
/etc/neutron/metadata_agent.ini

[DEFAULT]
# ...
nova_metadata_host = 192.168.40.61
metadata_proxy_shared_secret = ducnm37
```

- Cấu hình network service:
```
/etc/nova/nova.conf

[DEFAULT]
# ...
core_plugin = ml2 (bật tính năng plug-in ml2 để quản lý network layer 2)
service_plugins = router (bật tính năng plug-in router)
allow_overlapping_ips = true (cho phép overlap ip)
transport_url = rabbit://openstack:ducnm37@192.168.40.61 (cấu hình truy cập queue RabbitMQ)
auth_strategy = keystone (cấu hình xác thực bằng keystone)
notify_nova_on_port_status_changes = true (thông báo tới compute khi network topologies có sự thay đổi về status)
notify_nova_on_port_data_changes = true (thông báo tới compute khi network topologies có sự thay đổi về data)

[database]
# ...
connection = mysql+pymysql://nova:ducnm37@192.168.40.61/nova (khai báo truy cập database)

[keystone_authtoken] (cấu hình truy cập service định danh cho neutron qua keystone)
# ...
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = ducnm37

[nova] (cấu hình cho phép nova kết nối tới neutron)
# ...
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = ducnm37

[oslo_concurrency] 
# ...
lock_path = /var/lib/neutron/tmp (cấu hình lock path)

[neutron] (cấu hình lấy các parameter, bật tính năng metadata proxy và mật khẩu để user neutron truy cập)
# ...
url = http://controller:9696
auth_url = http://controller:500
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

- Cấu hình ml2 plug-in (Modular Layer 2 plug-in) xây dựng hạ tầng lớp 2 cho các instances:
```
/etc/neutron/plugins/ml2/ml2_conf.ini

[ml2]
# ...
type_drivers = flat,vlan,vxlan (bật service flat, vlan, vxlan network)
tenant_network_types = vxlan (self-service sử dụng vxlan)
mechanism_drivers = linuxbridge,l2population (bật cơ chế Linux bridge và layer-2 population )
extension_drivers = port_security (port bảo vệ)

[ml2_type_flat]
# ...
flat_networks = provider (provider sử dụng cơ chế flat)

[securitygroup]
# ...
enable_ipset = true (bật ipset để tăng tính hiệu quả của sercurity group)
```

- Cấu hình Linux bridge agent, xây dựng hạ tầng mạng ảo lớp 2 cho các instances và xử lý các sercurity group:
```
/etc/neutron/plugins/ml2/linuxbridge_agent.ini

[linux_bridge]
physical_interface_mappings = provider:eth0 (cấu hình ánh xạ mạng ảo vào card mạng vật lý)

[vxlan]
enable_vxlan = True (bật tính lớp mạng vxlan)
local_ip = 10.10.10.61 (ip sử dụng để quản lý trong local trên controller) 
l2_population = true 

[securitygroup] (cấu hình security groups, Linux bridge, iptables, firewall)
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

- Cấu hình dịch vụ dhcp cho mạng ảo:
```
/etc/neutron/dhcp_agent.ini

[DEFAULT] (cấu hình sử dụng linux bridge, DHCP, cho phép các instance truy cập metadata thông qua provider network)
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true (bật tính năng cô lập metadata với việc cấp ip dhcp)
```

- Cấu hình layer-3 agent, định tuyến và NAT cho self-service
```
/etc/neutron/l3_agent.ini

[DEFAULT] (cấu hình routing và NAT cho self-service sử dụng liluxbridge)
# ...
interface_driver = linuxbridge
```
