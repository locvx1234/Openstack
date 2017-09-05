## Block Storage service

OpenStack Block service (cinder) cung cấp một cơ sở hạ tầng cho việc quản lý các volume và tương tác với OpenStack Compute để cũng cấp volume cho instance. 

Service cũng cho phép quản lý các volume snapshot, volume type. 

Block Storage bao gồm các thành phần sau :

**cinder-api** : Chấp nhận các API request và định tuyến chúng tới `cinder-volume` 

**cinder-volume** : Tương tác trực tiếp với Block Storage service và các tiến trình như `cinder-scheduler`. Nó cũng tương tác với các tiến trình qua `Message queue`. 

**cinder-scheduler daemon** : Lựa chọn node cung cấp storage tối ưu để tạo volume. Một thành phần tương tự là `nova-scheduler`

**cinder-backup daemon** : `cinder-backup` service cung cấp backup volume, giống như `cinder-volume`, nó có thể tương tác với nhiều storage provider.

**Messaging queue** : định tuyến thông tin giữa các tiếng trình Block Storage


### Cài đặt và cầu hình trên node Controller

- Tạo database cinder và phân quyền 

```
# mysql

MariaDB [(none)]> CREATE DATABASE cinder;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'locvx1234';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'locvx1234';
```

- Sử dụng `admin-openrc`

```
$ . admin-openrc
```
  
- Tạo user 

```
$ openstack user create --domain default --password-prompt cinder

+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 8aae9b2286534f09b0591cfc1bd12dc4 |
| name                | cinder                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

- Add role 

```
$ openstack role add --project service --user cinder admin
```

- Tạo 2 entity service là cinder2 và cinder3 

```
$ openstack service create --name cinderv2 \
--description "OpenStack Block Storage" volumev2

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | 681e2263f0694c3fb692dbaa00970be0 |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+
```

```
$ openstack service create --name cinderv3 \
--description "OpenStack Block Storage" volumev3

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | c2dd4afc33ca4550b676467f40be0e44 |
| name        | cinderv3                         |
| type        | volumev3                         |
+-------------+----------------------------------+
```

- Tạo các endpoint Block Storage service API 

```
$ openstack endpoint create --region RegionOne \
volumev2 public http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | f956afbde61e467488d4e07e4e63120e         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 681e2263f0694c3fb692dbaa00970be0         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
volumev2 internal http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | d53da086af6f4e4d9c3b539c8af45807         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 681e2263f0694c3fb692dbaa00970be0         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
volumev2 admin http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 58d0b5e402ce4054a7ced5859bcafde1         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 681e2263f0694c3fb692dbaa00970be0         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
volumev3 public http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 772f9daaa7734a2eb6389fe2f44999e5         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | c2dd4afc33ca4550b676467f40be0e44         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
volumev3 internal http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | e3bef7b1ae104006bd095e8b5a2f7606         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | c2dd4afc33ca4550b676467f40be0e44         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
volumev3 admin http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 7c5bf8801f954f73aa08a861a0ad4d47         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | c2dd4afc33ca4550b676467f40be0e44         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+
```

- Cài đặt các package 

```
# apt install cinder-api cinder-scheduler
```

- Edit file `/etc/cinder/cinder.conf` :

```
# cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.orig 
# vi /etc/cinder/cinder.conf
``` 

```
[database]
# ...
connection = mysql+pymysql://cinder:locvx1234@controller/cinder

[DEFAULT]
# ...
transport_url = rabbit://openstack:locvx1234@controller

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
username = cinder
password = locvx1234

my_ip = 10.10.10.10

[oslo_concurrency]
# ...
lock_path = /var/lib/cinder/tmp

```

- Đăng ký database Block Storage 

```
# su -s /bin/sh -c "cinder-manage db sync" cinder
```

- Edit file `/etc/nova/nova.conf`

```
[cinder]
os_region_name = RegionOne		# line 4029
```

- Hoàn tất cài đặt 

```
# service nova-api restart

# service cinder-scheduler restart
# service apache2 restart
```

### Cài đặt và cấu hình một node storage 

Phần này mô tả cách cài đặt và cấu hình node storage cho service Block Storage.

Thực hiện các bước trên node Storage 

- Cài đặt 

```
# apt install lvm2
```

Một vài distro đã hỗ trợ sẵn LVM 

- Tạo LVM physical volume `/dev/sdb`

```
root@blockstorage:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created
```

- Tạo LVM volume group `cinder-volumes`

```
root@blockstorage:~# vgcreate cinder-volume /dev/sdb
  Volume group "cinder-volume" successfully created
```

- Thêm filter trong section devices ở file `/etc/lvm/lvm.conf`

```
devices {
	....
	filter = ["a/sdb/", "r/.*/"]
```
- Edit file `/etc/cinder/cinder.conf` 

Bổ sung các tham số sau 

```
transport_url = rabbit://openstack:locvx1234@controller
my_ip = 10.10.10.101
enabled_backends = lvm
glance_api_servers = http://controller:9292

[database]
# ...
connection = mysql+pymysql://cinder:locvx1234@controller/cinder

[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = locvx1234

[lvm]
# ...
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = tgtadm

[oslo_concurrency]
# ...
lock_path = /var/lib/cinder/tmp
```

- Restart các dịch vụ
```
root@blockstorage:~# service tgt restart
root@blockstorage:~# service cinder-volume restart
```

- List 

```
root@blockstorage:~# . admin-openrc
root@blockstorage:~# openstack volume service list
+------------------+------------------+------+---------+-------+----------------------------+
| Binary           | Host             | Zone | Status  | State | Updated At                 |
+------------------+------------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller       | nova | enabled | up    | 2017-09-05T11:15:41.000000 |
| cinder-volume    | blockstorage@lvm | nova | enabled | up    | 2017-09-05T11:15:40.000000 |
+------------------+------------------+------+---------+-------+----------------------------+
```




:arrow_right: [Additional services]()
