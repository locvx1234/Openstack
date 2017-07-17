:arrow_left: [Identity](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Identity.md)

## Image service (Glance)

Image service cho phép các user discover, register và retrive các image VM. Bạn có thể lưu trữ các image VM thông qua Image service từ các hệ thống file đơn giản cho đến các hệ thống lưu trữ hướng đối tượng như OpenStack Object Storage.

Để đơn giản, guide này mô tả cấu hình Image service sử dụng file back end, nó sẽ upload và lưu trữ trong một thư mục trên node controller. Mặc định, thư mục này là /var/lib/glance/images/. 

OpenStack Image service là trung tâm của IaaS. Nó chấp nhận các API request cho disk hoặc server image và các metadata từ end user hoặc các thành phần của OpenStack Compute. Nó cũng hỗ trợ lưu trữ disk hoặc server image trên nhiều loại repository bao gồm cả OpenStack Object Storage.

OpenStack Image service bao gồm các thành phần : 

- **glance-api** : chấp nhận các API cho khai thác, truy xuất và lưu trữ.

- **glance-registry** : truy xuất, xử lý, và lưu trữ metadata về các image. Metadata bao gồm các item như size và type. 

- **Database** : lưu trữ image và bạn có thể chọn database tùy thuộc vào sở thích. Hầu hết đều sử dụng MySQL hoặc SQLite.

- **Storage repository cho các file image** : Nhiều loại repository đã hỗ trợ bao gồm file system, Object Storage, RADOS block device, VMware datastore và HTTP. Lưu ý rằng một vài database chỉ hỗ trợ chế độ read-only

- **Metadata definition service** : Một API chung cho vendor, admin, service và user để định nghĩa metadata. Metadata này có thể là khác loại resource như image, artifact, volume, flavor và aggregate. 

### Cài đặt và cấu hình 

Phần này mô tả các bước cài đặt Image servive (glance) trên node Controller. Để đơn giản, cấu hình này lưu trữ image trên local file system.

- Tạo database glance 

```
# mysql
	
MariaDB [(none)]> CREATE DATABASE glance;
	
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
	IDENTIFIED BY 'GLANCE_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
	IDENTIFIED BY 'GLANCE_DBPASS';
```

		
- Sử dụng các biến trong file `admin-openrc` ở phần Identity service

```
$ . admin-openrc
```
	
File `admin-openrc` : 

```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=locvx1234
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

- Tạo glance user :

```
$ openstack user create --domain default --password-prompt glance
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 96a931938eb14606b1e3d74b290061e7 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```	
	
- Thêm admin role cho user `glance` và project `service`

```
$ openstack role add --project service --user glance admin
```

- Tạo glance service entity : 

```
$ openstack service create --name glance \
  --description "OpenStack Image" image
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 0987183d8947439cab6d9442eecb6457 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+

```

- Tạo các Image service API endpoint : 

```
$ openstack endpoint create --region RegionOne \
image public http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 54b3a1f204f04055b1876d134e2964bd |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0987183d8947439cab6d9442eecb6457 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

```
 
```
$ openstack endpoint create --region RegionOne \
 image internal http://controller:9292
```		
		
```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9af3dea1dcb442edadc45dfcda250414 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0987183d8947439cab6d9442eecb6457 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

```	

```
$ openstack endpoint create --region RegionOne \
 image admin http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 57f0e27b5def450e9395bf42589f19ae |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0987183d8947439cab6d9442eecb6457 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```		


- Cài packages

```
# apt install glance -y
```
	
- Edit file `/etc/glance/glance-api.conf`

```
# cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.orig 
# vi /etc/glance/glance-api.conf
```

```
[database]
# ...
connection = mysql+pymysql://glance:locvx1234@controller/glance   # line 1826

```
	
```
[keystone_authtoken]			# line 3282		
# ...	
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = locvx1234
[paste_deploy]
# ...
flavor = keystone                 # line 4268

[glance_store]					  # line 1915
# ...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```	
	
- Edit file `/etc/glance/glance-registry.conf`

```
# cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.orig 
# vi /etc/glance/glance-registry.conf 
```	

```
[database]
# ...
connection = mysql+pymysql://glance:locvx1234@controller/glance  # line 1116

[keystone_authtoken]											# line 1205
# ...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = locvx1234
[paste_deploy]
# ...
flavor = keystone

```

- Đăng ký vào Image service database: 

```
# su -s /bin/sh -c "glance-manage db_sync" glance
```

- Restart Image service 

```
# service glance-registry restart
# service glance-api restart
```

- Test 

Sử dụng CirrOS, một distro nhỏ nhẹ cho việc test Image service 

```
$ . admin-openrc
$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
```

- Upload image để Image service sử dụng định dạng disk là QCOW2, bare container và public để tất cả các project có thể truy cập nó: 

```
$ openstack image create "cirros" \
	--file cirros-0.3.5-x86_64-disk.img \
	--disk-format qcow2 --container-format bare \
	--public
```
		
```
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2017-07-03T04:18:54Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/0fc50da8-b841-4ddd-a457-49a639daa7a0/file |
| id               | 0fc50da8-b841-4ddd-a457-49a639daa7a0                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | 210387a5dc824ac2aaa93fdee42784ed                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-07-03T04:18:54Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```


- Liệt kê các image 

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

:arrow_right: [Compute service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Compute.md)
