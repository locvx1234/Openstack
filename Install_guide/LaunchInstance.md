:arrow_left: [Additional Service]()

## Launch an instance 

Phần này sẽ tạo các mạng ảo để hỗ trợ việc tạo các instance. 

Networking option 1 bao gồm 1 provider (external) network với 1 instance. 

Networking option 2 cũng bao gồm 1 provider (external) network với một instance và một self-service (private) network với một instance sử dụng nó. 

### Tạo provider network 

![Option 1 - overview]()

![Option 1 - connectivity]()

- Trên node Controller, sử dụng các biến trong file `admin-openrc`

```
$ . admin-openrc
```

- Tạo network 

```
$ openstack network create --share --external \
--provider-physical-network provider \
--provider-network-type flat provider
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        | nova                                 |
| created_at                | 2017-07-04T10:15:28Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | bdba20c0-ca52-4e51-9b69-f73719027862 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| project_id                | 210387a5dc824ac2aaa93fdee42784ed     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 6                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   | 8fca315b-19fc-4962-ab18-20ae39cebb5c |
| updated_at                | 2017-07-04T10:20:32Z                 |
+---------------------------+--------------------------------------+
```

- Sửa file `/etc/neutron/plugins/ml2/ml2_conf.ini` 

```
[ml2_type_flat]
flat_networks = provider
```

- Sửa file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini:`
 
```
[linux_bridge]
physical_interface_mappings = provider:eth0
```

- Tạo subnet 

```
$ openstack subnet create --network provider \
--allocation-pool start=START_IP_ADDRESS,end=END_IP_ADDRESS \
--dns-nameserver DNS_RESOLVER --gateway PROVIDER_NETWORK_GATEWAY \
--subnet-range PROVIDER_NETWORK_CIDR provider
```

```
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.169.155-192.168.169.253      |
| cidr              | 192.168.0.0/16                       |
| created_at        | 2017-07-04T10:20:32Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.0.254                        |
| host_routes       |                                      |
| id                | 8fca315b-19fc-4962-ab18-20ae39cebb5c |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | provider                             |
| network_id        | bdba20c0-ca52-4e51-9b69-f73719027862 |
| project_id        | 210387a5dc824ac2aaa93fdee42784ed     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-04T10:20:32Z                 |
+-------------------+--------------------------------------+
```

### Self-service network 

Self-service network kết nối với hạ tầng physical network qua NAT/ Mạng này bao gồm một DHCP server để cung cấp IP cho các instance. Một instance trên mạng này có thể tự động truy cập external network như Internet. 

Tuy nhiên để truy cập vào một instance trên network này từ external network như Internet thì cần một floating IP address.

*Warning* : Phải tạo provider network trước khi tạo self-service network.

![option 2 - overview]()

![option 2 - connectivity]

#### Tạo self-service network

- Trên node Controller, sử dụng file `demo-openrc`

```
$ . demo-openrc
```

- Tạo mạng 

```
$ openstack network create selfservice
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-07-06T09:48:55Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 0b388db3-98fb-4e14-b2e2-35a28a69da23 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| mtu                       | 1450                                 |
| name                      | selfservice                          |
| port_security_enabled     | True                                 |
| project_id                | b167aaf8ce33471d9683988868bbd048     |
| provider:network_type     | None                                 |
| provider:physical_network | None                                 |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2017-07-06T09:48:55Z                 |
+---------------------------+--------------------------------------+
```

- Tạo subnet trên network

```
$ openstack subnet create --network selfservice \
--dns-nameserver DNS_RESOLVER --gateway SELFSERVICE_NETWORK_GATEWAY \
--subnet-range SELFSERVICE_NETWORK_CIDR selfservice
```

```
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.10.10.2-10.10.10.254              |
| cidr              | 10.10.10.0/24                        |
| created_at        | 2017-07-06T09:56:56Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 10.10.10.1                           |
| host_routes       |                                      |
| id                | f4163933-bdcb-4ccf-9016-6528ab28efeb |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | selfservice                          |
| network_id        | 0b388db3-98fb-4e14-b2e2-35a28a69da23 |
| project_id        | b167aaf8ce33471d9683988868bbd048     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-06T09:56:56Z                 |
+-------------------+--------------------------------------+
```

### Tạo Router 

Self-service network kết nối với provider network sử dụng một vitual router để thực hiện NAT 2 chiều. 

Mỗi router chứa một interface trên ít nhất một self-service network và một gateway trên một provider network. 

Provider network phải bao gồm router `external` để cho phép các router self-service sử dụng nó để kết nối với external network như Internet. 

- Sử dụng `admin-openrc` để admin có quyền truy cập CLI của admin 

```
$ . admin-openrc
```

- Sử dụng `demo-openrc` để truy cập bằng quyền của user 

```
$ . demo-openrc
```

- Tạo router 

```
$ openstack router create router
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2017-07-06T11:26:17Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | 6c01ad2d-d7da-4932-bded-b4f84d264962 |
| name                    | router                               |
| project_id              | b167aaf8ce33471d9683988868bbd048     |
| revision_number         | None                                 |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2017-07-06T11:26:17Z                 |
+-------------------------+--------------------------------------+
```

- Thêm self-service network như một interface trên router 

```
$ neutron router-interface-add router selfservice

Added interface 2cb9bf8a-5bb6-4196-9164-13c541528340 to router router.
```

- Đặt gateway trên provider network trên router 

```
$ neutron router-gateway-set router provider

Set gateway for router router
```

### Verify operation 

```
$ . admin-openrc

$ ip netns
qrouter-6c01ad2d-d7da-4932-bded-b4f84d264962 (id: 2)
qdhcp-0b388db3-98fb-4e14-b2e2-35a28a69da23 (id: 3)
qdhcp-bdba20c0-ca52-4e51-9b69-f73719027862 (id: 1)
qdhcp-547d6357-86d0-4e43-8555-5acbe9eb244c (id: 0)

$ neutron router-port-list router

+--------------------------------------+------+----------------------------------+-------------------+-----------------------------------------+
| id                                   | name | tenant_id                        | mac_address       | fixed_ips                               |
+--------------------------------------+------+----------------------------------+-------------------+-----------------------------------------+
| 2cb9bf8a-5bb6-4196-9164-13c541528340 |      | b167aaf8ce33471d9683988868bbd048 | fa:16:3e:f9:58:02 | {"subnet_id": "f4163933-bdcb-           |
|                                      |      |                                  |                   | 4ccf-9016-6528ab28efeb", "ip_address":  |
|                                      |      |                                  |                   | "10.10.10.1"}                           |
| 4d8e5189-b942-4b09-8406-c22a1bf35f26 |      |                                  | fa:16:3e:63:6d:d5 | {"subnet_id": "8fca315b-                |
|                                      |      |                                  |                   | 19fc-4962-ab18-20ae39cebb5c",           |
|                                      |      |                                  |                   | "ip_address": "192.168.169.164"}        |
+--------------------------------------+------+----------------------------------+-------------------+-----------------------------------------+
```


### Tạo m1.nano flavor

Flavor là một vitual hardware template, nó bao gồm memory, cpu, disk, network,...

Flavor mặc định nhỏ nhất là 512 MB RAM với mỗi instance. Trong môi trường lab với Compute node nhỏ hơn 4GB thì chỉ cần 64 MB mỗi instance thôi cũng được và chỉ sử dụng với CirrOS để test.

```
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano

+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| access_project_ids         | None    |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```

### Tạo một cặp key

Hầu hết các cloud image đều hỗ trợ xác thực bằng public key, nó sẽ an toàn và tiện dụng hơn là xác thực bằng password.

Add một public key vào Compute service : 

```
$ . demo-openrc
$ ssh-keygen -q -N ""
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey

+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 4d:16:dc:91:03:07:bd:1f:15:f0:02:9e:72:43:3e:3e |
| name        | mykey                                           |
| user_id     | 75a4a173328b467a81dc6e3fe56af74b                |
+-------------+-------------------------------------------------+
```

- Xem cặp key 

```
$ openstack keypair list

+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | 4d:16:dc:91:03:07:bd:1f:15:f0:02:9e:72:43:3e:3e |
+-------+-------------------------------------------------+
```

### Thêm các rule security group 

Mặc định, security group `default` chấp nhận cho tất cả instance và bao gồm cả các rule firewall mà từ chối remote access vào các instance. Với CirrOS, recomment là cho phép ít nhất giao thức ICMP và SSH.

#### Thêm rule vào security group `default` : 

- Cho phép ping vào instance:

```
$ openstack security group rule create --proto icmp default

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-07T04:41:11Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 3d9e859c-4ffe-44a2-be6c-11d2650f4864 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | b167aaf8ce33471d9683988868bbd048     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 49038108-6b04-4c95-be98-d94ab2d5dfaf |
| updated_at        | 2017-07-07T04:41:11Z                 |
+-------------------+--------------------------------------+
```

- Cho phép ping vào instance:

```
$ openstack security group rule create --proto tcp --dst-port 22 default

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-07T04:42:55Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 22e2e291-6604-40b8-ae82-c9071e2be070 |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | b167aaf8ce33471d9683988868bbd048     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 49038108-6b04-4c95-be98-d94ab2d5dfaf |
| updated_at        | 2017-07-07T04:42:55Z                 |
+-------------------+--------------------------------------+
```

### Launch instance

Thay thế `PROVIDER_NET_ID` bằng ID của `provider`
```
$ openstack server create --flavor m1.nano --image cirros \
--nic net-id=PROVIDER_NET_ID --security-group default \
--key-name mykey provider-instance

+-----------------------------+-----------------------------------------------+
| Field                       | Value                                         |
+-----------------------------+-----------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                        |
| OS-EXT-AZ:availability_zone |                                               |
| OS-EXT-STS:power_state      | NOSTATE                                       |
| OS-EXT-STS:task_state       | scheduling                                    |
| OS-EXT-STS:vm_state         | building                                      |
| OS-SRV-USG:launched_at      | None                                          |
| OS-SRV-USG:terminated_at    | None                                          |
| accessIPv4                  |                                               |
| accessIPv6                  |                                               |
| addresses                   |                                               |
| adminPass                   | kRt4k9hRT2M3                                  |
| config_drive                |                                               |
| created                     | 2017-07-07T06:44:16Z                          |
| flavor                      | m1.nano (0)                                   |
| hostId                      |                                               |
| id                          | b484983f-fd2d-47e1-8cb7-5e2691b246d9          |
| image                       | cirros (0fc50da8-b841-4ddd-a457-49a639daa7a0) |
| key_name                    | mykey                                         |
| name                        | provider-instance                             |
| progress                    | 0                                             |
| project_id                  | b167aaf8ce33471d9683988868bbd048              |
| properties                  |                                               |
| security_groups             | name='default'                                |
| status                      | BUILD                                         |
| updated                     | 2017-07-07T06:44:16Z                          |
| user_id                     | 75a4a173328b467a81dc6e3fe56af74b              |
| volumes_attached            |                                               |
+-----------------------------+-----------------------------------------------+
```

- Check status instance 

```
$ openstack server list

+--------------------------------------+-------------------+--------+--------------------------+------------+
| ID                                   | Name              | Status | Networks                 | Image Name |
+--------------------------------------+-------------------+--------+--------------------------+------------+
| b484983f-fd2d-47e1-8cb7-5e2691b246d9 | provider-instance | ACTIVE | provider=192.168.169.156 | cirros     |
+--------------------------------------+-------------------+--------+--------------------------+------------+
```

### Truy cập instance sử dụng virtual console 

Xác định URL Virtual Network Computing (VNC) của instance để truy cập từ trình duyệt :

```
$ openstack console url show provider-instance

+-------+---------------------------------------------------------------------------------+
| Field | Value                                                                           |
+-------+---------------------------------------------------------------------------------+
| type  | novnc                                                                           |
| url   | http://controller:6080/vnc_auto.html?token=d298c4be-9f7a-473b-a9d4-77696c044b1c |
+-------+---------------------------------------------------------------------------------+
```

Từ giao diện console trên trình duyệt, bạn có thể thử ping ra default-gateway của provider network và ping ra Internet 

### Launch instance on the self-service network

```
$ . demo-openrc

$ openstack server create --flavor m1.nano --image cirros \
--nic net-id=SELFSERVICE_NET_ID --security-group default \
--key-name mykey selfservice-instance

+-----------------------------+-----------------------------------------------+
| Field                       | Value                                         |
+-----------------------------+-----------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                        |
| OS-EXT-AZ:availability_zone |                                               |
| OS-EXT-STS:power_state      | NOSTATE                                       |
| OS-EXT-STS:task_state       | scheduling                                    |
| OS-EXT-STS:vm_state         | building                                      |
| OS-SRV-USG:launched_at      | None                                          |
| OS-SRV-USG:terminated_at    | None                                          |
| accessIPv4                  |                                               |
| accessIPv6                  |                                               |
| addresses                   |                                               |
| adminPass                   | oVjGerea7T7A                                  |
| config_drive                |                                               |
| created                     | 2017-07-07T08:12:46Z                          |
| flavor                      | m1.nano (0)                                   |
| hostId                      |                                               |
| id                          | 57ee3bf0-b06b-400c-a606-e15b7ede8801          |
| image                       | cirros (0fc50da8-b841-4ddd-a457-49a639daa7a0) |
| key_name                    | mykey                                         |
| name                        | selfservice-instance                          |
| progress                    | 0                                             |
| project_id                  | b167aaf8ce33471d9683988868bbd048              |
| properties                  |                                               |
| security_groups             | name='default'                                |
| status                      | BUILD                                         |
| updated                     | 2017-07-07T08:12:46Z                          |
| user_id                     | 75a4a173328b467a81dc6e3fe56af74b              |
| volumes_attached            |                                               |
+-----------------------------+-----------------------------------------------+
 
```