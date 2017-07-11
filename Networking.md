## Networking service

OpenStack Networking (neutron) cho phép tạo và gắn các interface được quản lý bởi các service OpenStack khác. Các plug-in có thể được implement để chứa các thiết bị và phần mềm mạng khác nhau, cung cấp tính linh hoạt cho kiến trúc và triển khai OpenStack.

Nó bao gồm các thành phần : 

- *neutron-server* : chấp nhận và định tuyến các API request tới plug-in thích hợp 
- *OpenStack Networking plug-ins and agents* : plug và unplug port, tạo network hoặc subnet và cung cấp địa chỉ IP. Các plug-in và các agent khác nhau tùy thuộc nhà cung cấp và công nghệ sử dụng trong cloud. OpenStack Networking cung cấp các plug-in và agent cho các Switch Cisco virtual và physic, NEC OpenFlow, Open vSwitch, Linux bridge và VMware NSX.

Các agent thông thường là L3 (layer 3), DHCP, và một plug-in agent.

- *Messaging queue* : được sử dụng để định tuyến thông tin giữa các neutron-server và các agent. Nó cũng hoạt động như một database để lưu trữ trạng thái cho cac plugin cụ thể


### Networking (neutron) concepts

OpenStack Networking (neutron) quản lý tất cả các khía cạnh mạng cho Virtual Networking Infrastructure và lớp truy cập Physical Networking Infrastructure. t. OpenStack Networking enable các project để tạo các topo mạng nâng cao, có thể bao gồm firewall, load balancer, VPN.

Networking cung cấp mạng, mạng con, bộ định tuyến như các đối tượng trừu tượng để giả lập các đối tượng vật lý tương ứng. 

Bất kỳ cấu hình Networking nào cũng có ít nhất một external network. Không như các network khác, external network thì không đơn thuần là một mạng ảo được định nghĩa. Thay vào đó, nó được xem như có một phần là physic. Địa chỉ IP trên extenal network có thể truy cập từ mạng bên ngoài.

Thêm vào đó, Networking cũng có ít nhất một hoặc nhiều internal network. Các SDN này kết nối trực tiếp với các VM. Chỉ các VM trên internal network hoặc trong các subnet được kết nối qua interface mới có thể kết nối được với các VM kết nối trực tiếp với mạng đó. 

Đối với mạng bên ngoài để truy cập vào VM và ngược lại thì cần đến các bộ định tuyến. Mỗi router có một cổng kết nối với external network và một hoặc nhiều interface để kết nối với mạng nội bộ. Giống như các bộ định tuyến vật lý, các subnet có thể truy cập các máy trên subnet khác cùng router, và các máy có thể truy cập ra mạng bên ngoài qua router. 

Bạn có thể phân bố địa chỉ IP trên external network đến các port trên internal network. Bất cứ khi nào một cái gì đó được kết nối với một subnet, kết nối đó được gọi đến một port. Ban có thể liên kết các địa chỉ IP external network với port VM, bằng cách này bên ngoài có thể truy cập vào các VM. 

Networking cũng hỗ trợ các nhóm bảo mật. Các nhóm bảo mật cho phép admin xác định firewall rule theo group. Một VM có thể thuộc về một hoặc nhiều group, và Networking áp dụng các rule trong các group để chặn hoặc bỏ chặn các port, range hoặc các loại traffic cho VM đó.

### Cài đặt và cấu hình node Controller 

- Tạo database và phân quyền 

```
# mysql

MariaDB [(none)] CREATE DATABASE neutron;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY 'locvx1234';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY 'locvx1234';

MariaDB [(none)]> exit
```

- Sử dụng các biến trong `admin-openrc`

```
$ . admin-openrc
```
	
- Tạo service credential  

```
$ openstack user create --domain default --password-prompt neutron
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | b2518c87406e4fefab44bcc2a78caf20 |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

- Thêm admin role cho user neutron 

```
$ openstack role add --project service --user neutron admin
```

- Tạo neutron service entity

```
$ openstack service create --name neutron \
--description "OpenStack Networking" network
	
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 85ea84e5ba1748e89a98769456fb24d4 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

- Tạo Networking service API endpoint : 

```
$ openstack endpoint create --region RegionOne \
network public http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6dc25668275b4fac9a34d9ca08a44843 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 85ea84e5ba1748e89a98769456fb24d4 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+


$ openstack endpoint create --region RegionOne \
network internal http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9407dc103a7741a5b0b966418f468ce2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 85ea84e5ba1748e89a98769456fb24d4 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
network admin http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8bb8f76eb88d45208731ab5bbf85cbf2 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 85ea84e5ba1748e89a98769456fb24d4 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

### Cấu hình networking 

Option 1 deploy kiến trúc đơn giản nhất chỉ hỗ trợ việc gắn các thực thể để cung cấp external network. Không private network, router hoặc floating IP address. 

User admin và những user có đặc quyền  có thể quản lý provider network.

Option 2 hỗ trợ option 1 với các service layer 3, giúp cho việc gắn các instance vào self-service (private) network. 

User demo và những user thường cũng có thể quản lý self-service network bao gồm router cung cấp kết nối giữa self-service và provider network.

Floating IP address cung cấp kết nối tới các instance sử dụng self-service từ external network như Internet.

### Networking Option 1: Provider networks

Cấu hình trên node Controller 

- Cài đặt các package

```
# apt install neutron-server neutron-plugin-ml2 \
neutron-linuxbridge-agent neutron-dhcp-agent \
neutron-metadata-agent
```

- Edit file `/etc/neutron/neutron.conf`

```
# cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.orig 
# vi /etc/neutron/neutron.conf
```


```
[database]
# ...
connection = mysql+pymysql://neutron:locvx1234@controller/neutron

core_plugin = ml2
service_plugins =

transport_url = rabbit://openstack:RABBIT_PASS@controller

auth_strategy = keystone

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
password = locvx1234

[DEFAULT]
# ...
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[nova]
# ...
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = locvx1234
```

- Cấu hình Modular Layer 2 (ML2) plug-in

```
# cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.orig
# vi /etc/neutron/plugins/ml2/ml2_conf.ini
```

```
[ml2]
# ...
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security

[ml2_type_flat]
# ...
flat_networks = provider

[securitygroup]
# ...
enable_ipset = true
```

- Cấu hình Linux bridge agent

```
# cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.orig 
# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

```
[linux_bridge]
physical_interface_mappings = provider:eth0

[vxlan]
enable_vxlan = false

[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

- Cấu hình DHCP agent 

```
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.orig
vi /etc/neutron/dhcp_agent.ini
```

```
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```


### Networking Option 2: Self-service networks

- Cài đặt các package : 

```
# apt install neutron-server neutron-plugin-ml2 \
neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
neutron-metadata-agent
```

- Edit file `/etc/neutron/neutron.conf`

```
# cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.orig 
# vi /etc/neutron/neutron.conf
```


```
[database]
# ...
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron  # line 765

[DEFAULT]
core_plugin = ml2											# line 30
service_plugins = router									# line 33
allow_overlapping_ips = true 								# line 91

transport_url = rabbit://openstack:RABBIT_PASS@controller 	# line 570

auth_strategy = keystone 									# line 27

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
password = locvx1234

[DEFAULT]
# ...
notify_nova_on_port_status_changes = true 	# line 99
notify_nova_on_port_data_changes = true		# line 103 

[nova]			# line 1111
# ...
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = locvx1234
```

- Cấu hình Modular Layer 2 (ML2) plug-in

```
# cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.orig
# vi /etc/neutron/plugins/ml2/ml2_conf.ini
```

```
[ml2]
# ...
type_drivers = flat,vlan, vxlan 	# line 121

tenant_network_types = vxlan		# line 126

mechanism_drivers = linuxbridge,l2population 	# line 130

extension_drivers = port_security		# line 135

[ml2_type_flat]
# ...
flat_networks = provider  				# line 171 

[ml2_type_vxlan]
# ...
vni_ranges = 1:1000						# line 224

[securitygroup]
# ...
enable_ipset = true						# line 248
```

- Cấu hình Linux bridge agent

```
# cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.orig 
# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

```
[linux_bridge]
physical_interface_mappings = provider:eth0	# line 155

[vxlan]
enable_vxlan = true							# line 188
local_ip = OVERLAY_INTERFACE_IP_ADDRESS		# line 209
l2_population = true 						# line 213

[securitygroup]
# ...
enable_security_group = true				# line 173
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver # line 168
``` 

- Cấu hình layer-3 agent

```
# cp /etc/neutron/l3_agent.ini /etc/neutron/l3_agent.ini.orig 
# vi /etc/neutron/l3_agent.ini
```

```
[DEFAULT]
# ...
interface_driver = linuxbridge
```

- Cấu hình DHCP agent `/etc/neutron/dhcp_agent.ini`

```
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

- Cấu hình metadata agent `/etc/neutron/metadata_agent.ini`

```
[DEFAULT]
# ...
nova_metadata_ip = controller
metadata_proxy_shared_secret = locvx1234
```

- Cấu hình Compute service sử dụng Networking service 
```
# vi/etc/nova/nova.conf
```

```
[neutron]				# line 6968
# ...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
service_metadata_proxy = true
metadata_proxy_shared_secret = locvx1234
```


### Hoàn tất cài đặt 

```
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

# service nova-api restart

# service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart
```

Với option 2, restart thêm layer-3 service 

```
# service neutron-l3-agent restart
```

### Cài đặt cấu hình trên node Compute 

Các lệnh thực hiện trên node Compute

- Cài package và sử file `/etc/neutron/neutron.conf`

```
# apt install neutron-linuxbridge-agent
# cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.orig 
# vi /etc/neutron/neutron.conf
```

```
[DEFAULT]
# ...
transport_url = rabbit://openstack:locvx1234@controller				# line 507

auth_strategy = keystone											# line 27

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
password = locvx1234 
```

- Cấu hình Networking 

Option 1 : Edit file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`

```
[linux_bridge]
physical_interface_mappings = provider:eth0

[vxlan]
enable_vxlan = false

[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

Option 2: Edit file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`

```
[linux_bridge]
physical_interface_mappings = provider:eth0

[vxlan]
enable_vxlan = true
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = true

[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

- Cấu hình service Compute sử dụng service Networking 

Edit file `/etc/nova/nova.conf`

```
[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = locvx1234
```

### Hoàn tất cài đặt 

```
# service nova-compute restart

# service neutron-linuxbridge-agent restart
```

### Verify 

Các lệnh thực hiện trên node Controller

```
$ . admin-openrc

$ openstack extension list --network
```

```
+-------------------------------------------------------+---------------------------+-------------------------------------------------------+
| Name                                                  | Alias                     | Description                                           |
+-------------------------------------------------------+---------------------------+-------------------------------------------------------+
| Default Subnetpools                                   | default-subnetpools       | Provides ability to mark and use a subnetpool as the  |
|                                                       |                           | default                                               |
| Network IP Availability                               | network-ip-availability   | Provides IP availability data for each network and    |
|                                                       |                           | subnet.                                               |
| Network Availability Zone                             | network_availability_zone | Availability zone support for network.                |
| Auto Allocated Topology Services                      | auto-allocated-topology   | Auto Allocated Topology Services.                     |
| Neutron L3 Configurable external gateway mode         | ext-gw-mode               | Extension of the router abstraction for specifying    |
|                                                       |                           | whether SNAT should occur on the external gateway     |
| Port Binding                                          | binding                   | Expose port bindings of a virtual port to external    |
|                                                       |                           | application                                           |
| agent                                                 | agent                     | The agent management extension.                       |
| Subnet Allocation                                     | subnet_allocation         | Enables allocation of subnets from a subnet pool      |
| L3 Agent Scheduler                                    | l3_agent_scheduler        | Schedule routers among l3 agents                      |
| Tag support                                           | tag                       | Enables to set tag on resources.                      |
| Neutron external network                              | external-net              | Adds external network attribute to network resource.  |
| Neutron Service Flavors                               | flavors                   | Flavor specification for Neutron advanced services    |
| Network MTU                                           | net-mtu                   | Provides MTU attribute for a network resource.        |
| Availability Zone                                     | availability_zone         | The availability zone extension.                      |
| Quota management support                              | quotas                    | Expose functions for quotas management per tenant     |
| HA Router extension                                   | l3-ha                     | Add HA capability to routers.                         |
| Provider Network                                      | provider                  | Expose mapping of virtual networks to physical        |
|                                                       |                           | networks                                              |
| Multi Provider Network                                | multi-provider            | Expose mapping of virtual networks to multiple        |
|                                                       |                           | physical networks                                     |
| Address scope                                         | address-scope             | Address scopes extension.                             |
| Neutron Extra Route                                   | extraroute                | Extra routes configuration for L3 router              |
| Subnet service types                                  | subnet-service-types      | Provides ability to set the subnet service_types      |
|                                                       |                           | field                                                 |
| Resource timestamps                                   | standard-attr-timestamp   | Adds created_at and updated_at fields to all Neutron  |
|                                                       |                           | resources that have Neutron standard attributes.      |
| Neutron Service Type Management                       | service-type              | API for retrieving service providers for Neutron      |
|                                                       |                           | advanced services                                     |
| Router Flavor Extension                               | l3-flavors                | Flavor support for routers.                           |
| Port Security                                         | port-security             | Provides port security                                |
| Neutron Extra DHCP opts                               | extra_dhcp_opt            | Extra options configuration for DHCP. For example PXE |
|                                                       |                           | boot options to DHCP clients can be specified (e.g.   |
|                                                       |                           | tftp-server, server-ip-address, bootfile-name)        |
| Resource revision numbers                             | standard-attr-revisions   | This extension will display the revision number of    |
|                                                       |                           | neutron resources.                                    |
| Pagination support                                    | pagination                | Extension that indicates that pagination is enabled.  |
| Sorting support                                       | sorting                   | Extension that indicates that sorting is enabled.     |
| security-group                                        | security-group            | The security groups extension.                        |
| DHCP Agent Scheduler                                  | dhcp_agent_scheduler      | Schedule networks among dhcp agents                   |
| Router Availability Zone                              | router_availability_zone  | Availability zone support for router.                 |
| RBAC Policies                                         | rbac-policies             | Allows creation and modification of policies that     |
|                                                       |                           | control tenant access to resources.                   |
| Tag support for resources: subnet, subnetpool, port,  | tag-ext                   | Extends tag support to more L2 and L3 resources.      |
| router                                                |                           |                                                       |
| standard-attr-description                             | standard-attr-description | Extension to add descriptions to standard attributes  |
| Neutron L3 Router                                     | router                    | Router abstraction for basic L3 forwarding between L2 |
|                                                       |                           | Neutron networks and access to external networks via  |
|                                                       |                           | a NAT gateway.                                        |
| Allowed Address Pairs                                 | allowed-address-pairs     | Provides allowed address pairs                        |
| project_id field enabled                              | project-id                | Extension that indicates that project_id field is     |
|                                                       |                           | enabled.                                              |
| Distributed Virtual Router                            | dvr                       | Enables configuration of Distributed Virtual Routers. |
+-------------------------------------------------------+---------------------------+-------------------------------------------------------+
```

```
$ openstack network agent list

+--------------------------------------+----------------+------------+-------------------+-------+-------+------------------------+
| ID                                   | Agent Type     | Host       | Availability Zone | Alive | State | Binary                 |
+--------------------------------------+----------------+------------+-------------------+-------+-------+------------------------+
| 0569f0a0-9eda-46ca-b719-2e91c6664fe9 | L3 agent       | controller | nova              | True  | UP    | neutron-l3-agent       |
| 1a646338-29e4-4548-aa4d-b447e4b3312f | Metadata agent | controller | None              | True  | UP    | neutron-metadata-agent |
| 220fb8c8-dd5e-4a72-8ce8-08e228424a18 | DHCP agent     | controller | nova              | True  | UP    | neutron-dhcp-agent     |
+--------------------------------------+----------------+------------+-------------------+-------+-------+------------------------+
```
