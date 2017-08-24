/etc/nova/nova.conf

```
[DEFAULT]
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
my_ip = 10.10.10.10
transport_url = rabbit://openstack:locvx1234@controller
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
force_dhcp_release=true
state_path=/var/lib/nova
enabled_apis=osapi_compute,metadata
log_dir=/var/log/nova

[api]
auth_strategy = keystone

[api_database]
connection = mysql+pymysql://nova:locvx1234@controller/nova_api

[cells]
enable=False

[cinder]
os_region_name = RegionOne

[database]
connection = mysql+pymysql://nova:locvx1234@controller/nova

[glance]
api_servers = http://controller:9292

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = locvx1234


[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = locvx1234
service_metadata_proxy = true
metadata_proxy_shared_secret = locvx1234


[oslo_concurrency]
lock_path = /var/lib/nova/tmp


[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = locvx1234

[vnc]
enabled = true
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
```

Cấu hình truy cập database
```
[api_database]
# ...
connection = mysql+pymysql://nova:locvx1234@controller/nova_api

[database]
# ...
connection = mysql+pymysql://nova:locvx1234@controller/nova
```

Cấu hình truy cập Rabbit
```
[DEFAULT]
# ...
transport_url = rabbit://openstack:locvx1234@controller
```

Cấu hình truy cập Keystone
```
[api]
# ...
# Có kiểu xác thực: sử dụng keystone và noathu2
# noathu2: được thiết kế chỉ để testing, vì nó không thực sự kiểm tra cridential. Noauth2 cung cấp một cridential có tính quản trị chi khi nếu có admin được chỉ định bằng username
# Chúng ta sử dụng keystone cho việc xác thực.
auth_strategy = keystone

[keystone_authtoken]
# ...
# Chúng ta phải cung cấp thông tin xác thực cho user nova để keystone thực hiện xác thực. NOVA_PASS là mật khẩu của user nova
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = locvx1234
```

Cấu hình biến my_ip để sử dụng cho các cấu hình khác
```
[DEFAULT]
# ...
# IP managerment trên node Controller
my_ip = 10.10.10.10
```

Enable hỗ trợ Networking
```
[DEFAULT]
# ...
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```

Cấu hình VNC proxy
```
[vnc]
# Virtual Network Computer (VNC) cung cấp remote desktop console truy cập đến các VM 
enabled = true

# ...
# IP/hostname server sẽ lắng nghe yêu cầu kết nối VNC (trong trường hợp này là node Controller)
vncserver_listen = $my_ip

# IP/hostname VNC console proxy.
# VNC proxy là một thành phần openstack cho phép dịch vụ compute truy cập tới các VM của họ thông qua VNC clients.
vncserver_proxyclient_address = $my_ip
```

















