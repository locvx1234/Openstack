:arrow_left: [Image service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Image.md)

# Compute service

## Overview Compute service

Sử dụng Compute để quản lý hệ thống cloud computing. OpenStack là một phần chính của IaaS. Các module chính được implement bằng python. 

OpenStack Compute tương tác với OpenStack Identity cho việc xác thực; với OpenStack Image để quản lý các image; với Dashboard để user và admin sử dụng GUI. 

OpenStack Compute có thể mở rộng theo chiều ngang và download các image để launch các instance. 

OpenStack Compute bao gồm các thành phần :

- **nova-api service** : hỗ trợ OpenStack Compute API, Amazon EC2 API, và một Admin API đặc biệt cho user có đặc quyền. 
- **nova-api-metadata service** : chấp nhận các metadata request từ instance. 
- **nova-compute service** : một daemon dùng để tạo và ngắt các instance qua các hypervisor API. Ví dụ :

	* XenAPI for XenServer/XCP
	* libvirt for KVM or QEMU
	* VMwareAPI for VMware

- **nova-placement-api service** : 
- **nova-scheduler service** :
- **nova-conductor module** : 
- **nova-cert module** : 
- **nova-consoleauth daemon** : 
- **nova-novncproxy daemon** : 
- **nova-spicehtml5proxy daemon** :
- **nova-xvpvncproxy daemon** :
- **The queue** : Thường sử dụng RabbitMQ
- **SQL database** : Lưu trữ các trạng thái build-time và run-time cho hạ tầng cloud, bao gồm :

	* Available instance types
	* Instances in use
	* Available networks
	* Projects

Trên lý thuyết, OpenStack Compute có thể hỗ trợ nhiều database có hỗ trợ SQLAlchemy. Các database thường sử dụng là SQLite3, MySQL, MariaDB, PosgreSQL.

## Cài đặt và cấu hình trên node Controller

Phần này sẽ mô tả cách cài đặt và cấu hình Compute service (Nova) trên node Controller

### Chuẩn bị

#### 1. Tạo các database : 

```	
# mysql
```

```
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> exit

```

#### 2. Sử dụng file [admin-openrc](https://github.com/locvx1234/Openstack/blob/master/open-rc/admin-openrc)

```
$ . admin-openrc
```

#### 3. Tạo các Compute service credential
	
- Tạo user `nova`

```
$ openstack user create --domain default --password-prompt nova
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | fc6bbc7b227047a7a49395b1c694f000 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

```	
	
- Thêm `admin` role 

```
$ openstack role add --project service --user nova admin
```
	
- Tạo `nova` service entity: 

```
$ openstack service create --name nova \
  --description "OpenStack Compute" compute
```
	  
```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 116ee53a5ec84eb79edc6b3b8c0a1058 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```


#### 4. Tạo Compute API service endpoint

```
$ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 03c61fc7b1ab460dae0562e11ae88e58 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 116ee53a5ec84eb79edc6b3b8c0a1058 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

```
$ openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a551a18aaa8b460787ff5d5e2ca47839 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 116ee53a5ec84eb79edc6b3b8c0a1058 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

```

```
$ openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 94a9cc76b4ae4aebada0e415696d03f9 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 116ee53a5ec84eb79edc6b3b8c0a1058 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```
	  
#### 5. Tạo Placement service user 

```
$ openstack user create --domain default --password-prompt placement
```
	  
```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 019ca3d1736e48468ebd628efe0319a0 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

```

#### 6. Thêm admin role vào user Placement

```
$ openstack role add --project service --user placement admin
```

#### 7. Tạo Placement API entry trong danh mục dịch vụ

```
$ openstack service create --name placement --description "Placement API" placement
```

```	
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | c882480e61a74244a648fa8dd6ee2639 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```

#### 8. Tạo Placement API endpoint

```
$ openstack endpoint create --region RegionOne placement public http://controller:8778
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 900d6b74ad694deaaf81bb4daa625ae2 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | c882480e61a74244a648fa8dd6ee2639 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```
$ openstack endpoint create --region RegionOne placement internal http://controller:8778
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 01c172d8d25346c1b1fd9273a0a92104 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | c882480e61a74244a648fa8dd6ee2639 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```
$ openstack endpoint create --region RegionOne placement admin http://controller:8778
```
	
```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 7fc82c392fd84444969d43ed46153256 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | c882480e61a74244a648fa8dd6ee2639 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```	

### Cài đặt và cấu hình các thành phần 

#### 1. Cài đặt các packages 

```
# apt install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler nova-placement-api -y
```

#### 2. Edit file `/etc/nova/nova.conf`

```
# cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
# vi /etc/nova/nova.conf
```
	
```
[DEFAULT]
# ...
my_ip = 10.10.10.10 						# line 1481

use_neutron = true											# line 2306 
firewall_driver = nova.virt.firewall.NoopFirewallDriver		# line 2465

transport_url = rabbit://openstack:locvx1234@controller				# line 3021

[api]
# ...
auth_strategy = keystone							# line 3085

[api_database]
# ...
connection = mysql+pymysql://nova:locvx1234@controller/nova_api 		# line 3379
[database]
# ...
connection = mysql+pymysql://nova:locvx1234@controller/nova 			# line 4396

[glance]
# ...
api_servers = http://controller:9292						# line 4953

[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = locvx1234

[placement]
# ...
os_region_name = RegionOne									# line 8129
auth_type = password										# line 8155
auth_url = http://controller:35357/v3						# line 8161
project_name = service										# line 8173
project_domain_name = Default								# line 8179
username = placement										# line 8199
user_domain_name = Default									# line 8205
password = locvx1234										# line 8208

[vnc]
enabled = true												# line 9704
# ...
vncserver_listen = $my_ip									# line 9727			
vncserver_proxyclient_address = $my_ip						# line 9739
```


#### 3. Đăng ký nove-api database

```
# su -s /bin/sh -c "nova-manage api_db sync" nova
```

#### 4. Đăng ký database `cell0`

```
# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```
	
#### 5. Tạo `cel11`	

```
# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
699fb0f3-e165-4ec9-9798-fe5925731a3a
```
	
#### 6. Đăng ký database `nova`

```
# su -s /bin/sh -c "nova-manage db sync" nova
```

#### 7. Liệt kê 2 cell đã đăng ký 

```
# nova-manage cell_v2 list_cells
```
	
```
+-------+--------------------------------------+
|  Name |                 UUID                 |
+-------+--------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |
| cell1 | 699fb0f3-e165-4ec9-9798-fe5925731a3a |
+-------+--------------------------------------+
```

### Khởi động lại các service

```
# service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart
```

## Cài đặt và cấu hình một node Compute 


Phần này sẽ mô tả các bước cài đặt và cấu hình Compute service trên node Compute. Service này hỗ trợ một vài hyvervisor để deloy các instance hoặc VM. Để đơn giản, cấu hình này sử dụng QEMU hypervisor với KVM extension trên node compute hỗ trợ tăng tốc phần cứng cho các máy ảo. 

### Cài đặt và cấu hình các thành phần 

#### 1. Cài đặt package nova-compute 

```
# apt install nova-compute
```
	
#### 2. Edit file `/etc/nova/nova.conf`

```
# cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
# vi /etc/nova/nova.conf
```
	
```
[DEFAULT]
# ...
transport_url = rabbit://openstack:locvx1234@controller

[api]
# ...
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
username = nova
password = locvx1234


[DEFAULT]
# ...
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS

[DEFAULT]
# ...
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[vnc]
# ...
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
# ...
api_servers = http://controller:9292

[oslo_concurrency]
# ...
lock_path = /var/lib/nova/tmp

[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = locvx1234

```
	
### Hoàn tất cài đặt 

- Xác định xem node Compute có tăng tốc độ phần cứng cho máy ảo hay không 

```
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```
	
Nếu lệnh trả về 1 hoặc lớn hơn, node Compute hỗ trợ tăng tốc mà không cần phải cấu hình thêm

Nếu lệnh trả về 0 thì phải cấu hình libvirt để sử dụng QEMU thay vì KVM 

```
# vi /etc/nova/nova-compute.conf
```
	
```
[libvirt]
# ...
virt_type = qemu
```
	
- Restart Compute service 

```
# service nova-compute restart
```
	
### Thêm node Compute vào cell database

**Note**: Các lệnh thực hiện trên node Controller 

- List các hypervisor 

```
$ . admin-openrc
$ openstack hypervisor list
```

```	
+----+---------------------+-----------------+--------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP      | State |
+----+---------------------+-----------------+--------------+-------+
|  1 | com1                | QEMU            | 10.10.10.100 | up    |
+----+---------------------+-----------------+--------------+-------+
```

- Discover compute hosts

```
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': 699fb0f3-e165-4ec9-9798-fe5925731a3a
Found 1 computes in cell: 699fb0f3-e165-4ec9-9798-fe5925731a3a
Checking host mapping for compute host 'com1': f00ad79a-8e68-476c-a237-25e743648a43
Creating host mapping for compute host 'com1': f00ad79a-8e68-476c-a237-25e743648a43
```

Khi bạn thêm một node Compute mới, bạn phải chạy lệnh `nova-manage cell_v2 discover_hosts` trên node Controller để đăng ký node mới. Hoặc cách khác là có thể đặt khoảng thời gian thích hợp trong file `/etc/nova/nova.conf`. 

```
# vi /etc/nova/nova.conf
	
[scheduler]
discover_hosts_in_cells_interval = 300							# line 8703
```
	
### Verify operation 

* Node * : Các lệnh thực hiện trên node Controller 

```
$ . admin-openrc
$ openstack compute service list
```

- List các thành phần của service 

```
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  3 | nova-consoleauth | controller | internal | enabled | up    | 2017-07-03T08:21:35.000000 |
|  4 | nova-scheduler   | controller | internal | enabled | up    | 2017-07-03T08:21:38.000000 |
|  5 | nova-conductor   | controller | internal | enabled | up    | 2017-07-03T08:21:43.000000 |
|  6 | nova-compute     | com1       | nova     | enabled | up    | 2017-07-03T08:21:37.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```	

- List các API endpoint trong Identity service 
	
```
$ openstack catalog list
```
	
```
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   admin: http://controller:35357/v3/    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
```

- List các image 

```
$ openstack image list
```
	
```
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 0fc50da8-b841-4ddd-a457-49a639daa7a0 | cirros | active |
+--------------------------------------+--------+--------+
```
- Kiểm tra các cell và placement API hoạt động thành công
```
# nova-status upgrade check
```

:arrow_right: [Networking service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Networking.md)