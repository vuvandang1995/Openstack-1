# File cấu hình config neutron

`/etc/neutron/metadata_agent.ini`: Cấu hình metadata

------------
Cấu hình Network Provider

- Cấu hình network service:
```
/etc/nova/nova.conf

[DEFAULT]
# ...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:RABBIT_PASS@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[database]
# ...
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron

[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS

[nova]
# ...
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS

[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp
```

- Cấu hình ml2 plug-in (Modular Layer 2 plug-in) xây dựng hạ tầng lớp 2 cho các instances:
```
/etc/neutron/plugins/ml2/ml2_conf.ini

[ml2]
# ...
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
flat_networks = provider
enable_ipset = true
```

- Cấu hình Linux bridge agent, xây dựng hạ tầng mạng ảo lớp 2 cho các instances và xử lý các sercurity group:
```
/etc/neutron/plugins/ml2/linuxbridge_agent.ini

[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME

[vxlan]
enable_vxlan = True
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = true

[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

- Cấu hình dịch vụ dhcp cho mạng ảo:
```
/etc/neutron/dhcp_agent.ini

[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

