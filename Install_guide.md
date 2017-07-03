
# Contents

## Overview 

Openstack là một nền tảng điện toán đám mây với mục đích đơn giản hóa việc thực hiện, khả năng mở rộng cao và tính năng phong phú. 

Tính đến thời điểm hiện tại là bản Release OpenStack thứ 15, có tới 1925 developer trên toàn thế giới tham gia đóng góp cho dự án này.

* Note : * Hướng dẫn này sử dụng để tìm hiểu về OpenStack chứ không phải để triển khai product

Sau khi làm quen với cài đặt cơ bản, cấu hình, xử lý sự cố các dịch vụ của OpenStack, các bước để triển khai product : 

- Xác định và implement các dịch vụ cốt lõi và các tùy chọn cần thiết để đáp ứng hiệu năng và dự phòng

- Tăng cường an ninh bằng các phương pháp như firewall, mã hóa, các policy dịch vụ 

- Sử dụng các công cụ như Ansible, Chef, Puppet, Salt để tự động triển khai và quản lý product.


### Example architecture 

Mô hình ví dụ này cần tối thiểu 2 node (host) để chạy một virtual machine hoặc instance.

Các service tùy chọn như Block Storage và Object Storage thì cần thêm các node bổ sung. 

Mô hình này là mô hình có cấu hình tối thiểu khác với mô hình triển khai product : 

- Các Networking agent được cài trên Node controller thay vì trên một node Network chuyên biệt

- Overlay (tunnel) traffic for self-service networks traverses the management network instead of a dedicated network.
 
Yêu cầu phần cứng :

![Hardware requirement]() 

Giải thích các thành phần 

#### Controller

Node Controller chạy Identity service, Image service, quản lý Compute, quản lý Networking, Networking agent và Dashboard. Nó cũng bao gồm hỗ trợ các dịch vụ như SQL database, message queue, NTP

Tùy chọn, Node Controller có thể chạy Block Storage, Object Storage, Orchestration, Telemetry. 

Node Controller cần ít nhất 2 network interface.

#### Compute 

Node Compute chạy phần hypervisor của Compute để điều khiển instance. Mặc định, Compute sử dụng KVM hypervisor. 

Node Compute cũng chạy một agent Networking service, nó sẽ connect các instance để ảo hóa network và cung cấp firewall cho các service.

Có thể deploy nhiều hơn 1 node Compute. Mỗi node tối thiểu 2 network interface.


#### Block Storage

Node tùy chọn: Block Storage chứa các disk sử dụng cho các dịch vụ Block Storage và Shared File System để cũng cấp cho instance.

Để đơn giản, traffic của service giữa các node compute và node này sử dụng chung với đường management. Còn trên môi trường product, nên triển khai một đường mạng riêng để tăng hiệu quả và an ninh.

Có thể deploy nhiều hơn 1 node Block Storage. Mỗi node tối thiểu 2 network interface.


#### Object Storage

Node tùy chọn : Object Storage chứa các disk sử dụng cho các dịch vụ Object Storage để sử dụng cho việc lưu trữ các account, containerm object.

Để đơn giản, traffic của service giữa các node compute và node này sử dụng chung với đường management. Còn trên môi trường product, nên triển khai một đường mạng riêng để tăng hiệu quả và an ninh.

Có thể deploy nhiều hơn 1 node Block Storage. Mỗi node tối thiểu 2 network interface.

Service này cần 2 node. Mỗi node tối thiểu 2 network interface. Có thể deploy nhiều hơn 2 node.

#### Networking

Có 2 mô hình 

##### Networking option 1: Mạng nhà cung cấp (Provider network)

Các service chủ yếu là layer 2 (bridging/switching) và VLAN. Về cơ bản, nó làm cầu nối các mạng ảo với các mạng vật lý trên cơ sở hạ tầng mạng vật lý chơ các service layer 3.

Warning: Option này không hỗ trợ cho mạng self-service (private), layer 3 (routing) service, và các service nâng cao như LBaaS, FWaaS.

![provider]()
##### Networking option 2: Mạng cá nhân (Self-service network)

Mô hình này cung cấp các service layer 3. Về cơ bản, nó sẽ định tuyến các mạng ảo tới các mạng vật lý sử dụng NAT. Ngoài ra, tùy chọn này cung cấp nền tảng cho các dịch vụ như LBaaS, FWaaS.

![self-service]()

## Environment

Phần này giải thích cách cấu hình Node Controller và 1 node Compute sử dụng Example architecture.

Hầu hết các môi trường bao gồm Identity, Image service, Compute, ít nhất một service networking và Dashboard, service Object Storage có thể sử dụng độc lập.

Yêu cầu tối thiểu trên CirrOS - một distribute Linux :

- Controller Node: 1 processor, 4 GB memory, 5 GB storage
- Compute Node: 1 processor, 2 GB memory, 10 GB storage

Nếu hiệu suất giảm khi enable thêm các service hoặc máy ảo, khi đó hãy xem xét bổ sung phần cứng.

Một phân vùng disk trên mỗi node được sử dụng cho các cài đặt cơ bản. Tuy nhiên, bạn nên xem xét LVM cho việc cài các service tùy chọn như Block Storage.

Cho lần cài đầu tiên, nhiều người sẽ chọn cách cài trên máy ảo vì những ưu điểm của nó, nhưng nó cũng làm giảm hiệu suất của instance.

### Security 

Các service OpenStack  hỗ trợ service theo các phương thức như password, policy, encryption. 

Để đơn giản quá trình cài đặt, guide này chỉ bảo mật theo password.

Password có thể tạo bằng tay, nhưng chuỗi kết nối database trong file cấu hình không được chứa ký tự đặc biệt như "@"/

Recommend sử dụng [pwgen](https://sourceforge.net/projects/pwgen/) hoặc chạy lệnh sau: 

	$ openssl rand -hex 10

Trong guide này sử dụng SERVICE_PASS để sử dụng cho password các service, SERVICE_DBPASS sử dụng cho password của database.

Đây là các biến pass 

![Password]()


### Host networking

Sau khi cài đặt OS trên mỗi node, bạn phải cấu hình network interface. 

![layer]()

![ip_planning]()

### Network Time Protocol (NTP)

Chúng ta cần phải đồng bộ thời gian cho các node. Để cài đặt NTP, ta sử dụng Chrony 

#### Trên Controller 

Cài đặt gói chrony cho NTP server

	# apt install -y chrony
Sao lưu file cấu hình của NTP Server

	# cp /etc/chrony/chrony.conf /etc/chrony/chrony.conf.orig
Cấu hình NTP server

'''
echo "server 1.vn.pool.ntp.org iburst 
server 0.asia.pool.ntp.org iburst 
server 3.asia.pool.ntp.org iburst

allow 10.10.10.0/24" >> /etc/chrony/chrony.conf
'''

Khởi động lại dịch vụ NTP trên Controller

	# systemctl restart chrony
Kiểm tra dịch vụ NTP đã hoạt động hay chưa

	# chronyc sources

Output :

'''
210 Number of sources = 8
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* time.vng.vn                   3   6    77    34  +1360us[+2949us] +/-   81ms
^- mta.khangthong.net            2   6    77    34    -42ms[  -42ms] +/-  260ms
^? 2001:b030:242b:ff00::1        0   6     0   10y     +0ns[   +0ns] +/-    0ns
^? makaki.miuku.net              0   6     0   10y     +0ns[   +0ns] +/-    0ns
^? x.ns.gin.ntt.net              0   6     0   10y     +0ns[   +0ns] +/-    0ns
^? 2001:df1:801:a005:3::1        0   6     0   10y     +0ns[   +0ns] +/-    0ns
^- send.mx.cdnetworks.com        2   6    77    34  +1829us[+1829us] +/-  207ms
^+ pontoon.latt.net              3   6    77    33   +490us[ +490us] +/-  152ms
'''	
	
Tất cả các node khác tham khảo node Controller để đồng bộ đồng hồ. Thực hiên tương tự trên tất cả các node khác.

### OpenStack packages


// TODO : chi tiết hơn 

Khai báo repo của OpenStack Ocata

	# apt install -y software-properties-common
	# add-apt-repository -y cloud-archive:ocata

Update Controller sau khi khai báo repos và khởi động lại máy chủ.

	# apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
	
Đăng nhập với vào máy chủ Controller và chuyển sang quyền root

	# su -

Cài đặt gói OpenStack Client

	# apt install -y python-openstackclient

	
### SQL database

Hầu hết các dịch vụ OpenStack đều cần SQL database để lưu trữ thông tin. Database thường chạy trên node Controller.

Chúng ta sẽ sử dụng MariaDB hoặc MySQL trong distro này, bên cạnh đó, các service OpenStack cũng hỗ trợ tốt với các SQL database khác như PostgreSQL.

Khai báo mật khẩu cho tài khoản root trong MariaDB # Phần này warning

	# echo mariadb-server-10.0 mysql-server/root_password locvx1234 | \
		debconf-set-selections
	# echo mariadb-server-10.0 mysql-server/root_password_again locvx1234 | \
		debconf-set-selections

Cài đặt gói của MariaDB

	# apt install -y mariadb-server python-pymysql

Tạo file cấu hình MariaDB dành cho các databasse OpenStack

	# vi /etc/mysql/mariadb.conf.d/99-openstack.cnf
	
	[mysqld]
	bind-address = 0.0.0.0

	default-storage-engine = innodb
	innodb_file_per_table = on
	max_connections = 4096
	collation-server = utf8_general_ci
	character-set-server = utf8



Restart lại dịch vụ 

	# systemctl restart mysql 
	
### Message queue (RabbitMQ)
	
OpenStack sử dụng Message queue để điều phối các hoạt động và thông tin trạng thái giữa các dịch vụ. Message queue thường chạy trên Controller. 

OpenStack hỗ trợ một vài Message Queue như RabbitMQ, Qpid, ZeroMQ. 
	
Cài đặt gói RabbitMQ

	# apt install rabbitmq-server
	
Thêm user openstack với pass locvx1234:

	# rabbitmqctl add_user openstack locvx1234

	Creating user "openstack" ...

Phân quyền trong rabbitmq đối với user `openstack` vừa tạo

	# rabbitmqctl set_permissions openstack ".*" ".*" ".*"

	Setting permissions for user "openstack" in vhost "/" ...

### Memcached 

Identity server cần sử dụng Memcached để cache các token. Memcached thường chạy trên Controller.

Cài gói memcached

	# apt install -y memcached python-memcache
	
Cấu hình memcached

	cp /etc/memcached.conf /etc/memcached.conf.orig
	
	sed -i 's/-l 127.0.0.1/-l 10.10.10.10/g' /etc/memcached.conf
	
Khởi động lại memcached

	systemctl enable memcached.service
	systemctl start memcached.service
		

## Identity service 

Identity service quản lý việc xác thực, cấp phép và một service catalog.

Identity service thường là service đầu tiên mà user tương tác. Một khi được xác thực, end user có thể sử dụng định danh của họ để truy cập các service khác của OpenStack.

Identity service cũng có thể tích hợp với một số hệ thống quản lý người dùng (như LDAP)

Các user và service có thể sử dụng các service catalog được quản lý bởi Identity service. Mỗi service có 1 hoặc nhiều endpoint thuộc 1 trong 3 loại : admin, internal hoặc public

Ví dụ : 

Public API network  có thể nhìn thấy từ Internet để khách hàng có thể quản lý các cloud của họ. 

Admin API network có thể bị giới hạn hoạt động trong các tổ chức quản lý hạ tầng cloud

Internal API network có thể bị giới hạn đối với máy chủ có chứa các dịch vụ OpenStack

Identity service chứa 3 thành phần : 

- * Server * : Một server tập trung cung cấp dịch vụ authentication và authorization sử dụng một RESTful interface
- * Drivers * : Các drive hoặc một service back end được tích hợp vào server tập trung. Chúng sử dụng để truy cập thông tin trong các repo bên ngoài OpenStack và có thể tồn tại trong hạ tầng mà OpenStack deploy (ví dụ SQL database, LDAP server)
- * Modules * :  Các module middleware trong OpenStack sử dụng Identity service

### Cài đặt và cấu hình 

Phần này mô tả các bước cài đặt và cấu hình Identity service - Keystone trên Controller. Cho mục đích mở rộng, cấu hình này deploy Fernet token và Apache HTTP server để xử lý yêu cầu.

Truy cập cơ sở dữ liệu: 

	# mysql -u root -p
	
Tạo keystone database:
	
	MariaDB [(none)]> CREATE DATABASE keystone default character set utf8;

Cấp quyền truy cập cho keystone database với KEYSTONE_DBPASS là locvx1234 :
	
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'locvx1234' WITH GRANT OPTION;
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'locvx1234' WITH GRANT OPTION;
	MariaDB [(none)]> FLUSH PRIVILEGES;
	
Exit database access 

	MariaDB [(none)]> exit
	
* Note : * Apache HTTP server với `mod_wsgi` để đáp ứng các request Identity service trên cổng 5000 và 35357 

Cài đặt gói keystone: 

	# apt install keystone -y
	# cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.orig
	
Edit file `/etc/keystone/keystone.conf` 

	[database]
	# ...
	connection = mysql+pymysql://keystone:locvx1234@controller/keystone   # line 713

	[token]
	# ...
	provider = fernet # line 2842
	
Đăng ký vào database Identity service

	# su -s /bin/sh -c "keystone-manage db_sync" keystone
	
Khởi tạo Fernet key repository:

	# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
	# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

	
Bootstrap Identity service: 

	# keystone-manage bootstrap --bootstrap-password locvx1234 \
		--bootstrap-admin-url http://controller:35357/v3/ \
		--bootstrap-internal-url http://controller:5000/v3/ \
		--bootstrap-public-url http://controller:5000/v3/ \
		--bootstrap-region-id RegionOne

### Cấu hình Apache HTTP server

	# vi /etc/apache2/apache2.conf
	
Option `ServerName` để chỉ ra tên node Controller

	ServerName controller

### Hoàn tất cài đặt 	

Restart Apache service và xóa SQLite database mặc định 

	# systemctl restart apache2
	# rm -f /var/lib/keystone/keystone.db

Cấu hình tài khoản admin : 

'''
$ export OS_USERNAME=admin
$ export OS_PASSWORD=locvx1234
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
	
'''

### Tạo domain, projects, users và các role

	$ openstack project create --domain default \
	--description "Service Project" service
	
'''	
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | eea7d969a33442bba2e93c4bd2296044 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
+-------------+----------------------------------+
'''

Những task thông thường (không phải admin) nên sử dụng một project và user không có nhiều quyền. Ví dụ :

- Tạo project 

	$ openstack project create --domain default \
	--description "Demo Project" demo	
	
'''
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | eea7d969a33442bba2e93c4bd2296044 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
+-------------+----------------------------------+

'''
	
- Tạo user 

	$ openstack user create --domain default \
	--password-prompt demo
	
# Note : mình dùng pass 000000
	
'''
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 75a4a173328b467a81dc6e3fe56af74b |
| name                | demo                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
'''
	
- Tạo user role 

	$ openstack role create user

'''
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 9ec4783f76504dd89fbb816e7438e54e |
| name      | user                             |
+-----------+----------------------------------+
'''

Thêm user role vào project và user demo 

	$ openstack role add --project demo --user demo user

### Xác nhận hoạt động 

Cần verify lại Identity service trước khi cài đặt các service khác. 

* Note : * Các lệnh thực hiện trên node Controller  	
	
Vì lý do bảo mật, disable cơ chế xác thực token tạm thời 

	# cp /etc/keystone/keystone-paste.ini /etc/keystone/keystone-paste.ini.orig
	# vi /etc/keystone/keystone-paste.ini
	
Xóa `admin_token_auth` trong các phần [pipeline:public_api], [pipeline:admin_api], và [pipeline:api_v3]

Unset biến môi trường y OS_AUTH_URL và OS_PASSWORD

	$ unset OS_AUTH_URL OS_PASSWORD
	
Request một xác thực token với user admin : 

	$ openstack --os-auth-url http://controller:35357/v3 \
	--os-project-domain-name default --os-user-domain-name default \
	--os-project-name admin --os-username admin token issue


'''
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-06-30T06:10:30+0000                                                                                                     |
| id         | gAAAAABZVd1GcSwhPb0TjXlV9Nq012GyeORRttY2Q0IssbWCQ6Lsuu8aIT1BAJLnjg6hVxy4ZxsSscwGZ8tJBoy95Q9q7aIoIq6WO9qAcdGEcMTJkoHzZRw6SPNu |
|            | _IjMlNGe23ev7heNcIv_RdXzUY-PMbz1LE4CePhNCC2vAdPBkqRG45BgXXI                                                                  |
| project_id | 210387a5dc824ac2aaa93fdee42784ed                                                                                             |
| user_id    | ee69e3d27c3c4e25b2e363b79f408e2f                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+
'''


Request một xác thực token với user demo : 

	$ openstack --os-auth-url http://controller:5000/v3 \
	--os-project-domain-name default --os-user-domain-name default \
	--os-project-name demo --os-username demo token issue

'''
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-06-30T06:12:03+0000                                                                                                     |
| id         | gAAAAABZVd2j7lJVeIPp1r2ZdMf38KC1kQHBiFqdH-_dCKN0RviP9fGsDeVLOdBwIRnMPmbIDQY5Ym4ZSlu1XCe9wdSSaoLRc4n8G_tMa9Ayz6_ldxau8DXL_UcQ |
|            | e90rXFCoYvaUHgAbDik1OdDZvA1GrYyPdSNtjQfjwgz73L8DxM6jBDwuII8                                                                  |
| project_id | b167aaf8ce33471d9683988868bbd048                                                                                             |
| user_id    | 75a4a173328b467a81dc6e3fe56af74b                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+
'''

### Script 

File `admin-openrc` :
	
'''
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
'''

File `demo-openrc`:

'''
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
'''
	
Sử dụng script : 

	$ . admin-openrc
	$ openstack token issue

	
## Image service (glance)

Image service cho phép các user discover, register và retrive các image VM. Bạn có thể lưu trữ các image VM thông qua Image service từ các hệ thống file đơn giản cho đến các hệ thống lưu trữ hướng đối tượng như OpenStack Object Storage.

Để đơn giản, guide này mô tả cấu hình Image service sử dụng file back end, nó sẽ upload và lưu trữ trong một thư mục trên node controller. Mặc định, thư mục này là /var/lib/glance/images/. 

OpenStack Image service là trung tâm của IaaS. Nó chấp nhận các API request cho disk hoặc server image và các metadata từ end user hoặc các thành phần của OpenStack Compute. Nó cũng hỗ trợ lưu trữ disk hoặc server image trên nhiều loại repository bao gồm cả OpenStack Object Storage.

OpenStack Image service bao gồm các thành phần : 

- * glance-api * : chấp nhận các API cho khám phá, truy xuất và lưu trữ.

- * glance-registry * : truy xuất, xử lý, và lưu trữ metadata về các image. Metadata bao gồm các item như size và type. 

- * Database * : lưu trữ image và bạn có thể chọn database tùy thuộc vào sở thích. Hầu hết đều sử dụng MySQL hoặc SQLite.

- * Storage repository cho các file image * : Nhiều loại repository đã hỗ trợ bao gồm file system, Object Storage, RADOS block device, VMware datastore và HTTP. Lưu ý rằng một vài database chỉ hỗ trợ chế độ read-only

- * Metadata definition service * : Một API chung cho vendor, admin, service và user để định nghĩa metadata. Metadata này có thể là khác loại resource như image, artifact, volume, flavor và aggregate. 

### Cài đặt và cấu hình 

Phần này mô tả các bước cài đặt Image servive (glance) trên node Controller. Để đơn giản, cấu hình này lưu trữ image trên local file system.

- Tạo database glance 

	# mysql
	
	MariaDB [(none)]> CREATE DATABASE glance;
	
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
		IDENTIFIED BY 'GLANCE_DBPASS';
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
		IDENTIFIED BY 'GLANCE_DBPASS';


		
- Sử dụng các biến trong file `admin-openrc` ở phần Identity service

	$ . admin-openrc
	
File `admin-openrc` : 

'''
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=locvx1234
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
'''

- Tạo glance user :

	$ openstack user create --domain default --password-prompt glance
	
'''
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

'''	
	
- Thêm admin role cho user `glance` và project `service`

	$ openstack role add --project service --user glance admin

- Tạo glance service entity : 

	$ openstack service create --name glance \
	  --description "OpenStack Image" image

'''
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 0987183d8947439cab6d9442eecb6457 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+

'''

- Tạo các Image service API endpoint : 

	$ openstack endpoint create --region RegionOne \
		image public http://controller:9292
 
 '''
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

 '''
 

	$ openstack endpoint create --region RegionOne \
	  image internal http://controller:9292
		
		
'''
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

'''	


	$ openstack endpoint create --region RegionOne \
	  image admin http://controller:9292

'''
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


'''		


- Cài packages

	# apt install glance -y
	
- Edit file `/etc/glance/glance-api.conf`

	# cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.orig 
	# vi /etc/glance/glance-api.conf

'''
[database]
# ...
connection = mysql+pymysql://glance:locvx1234@controller/glance   # line 1826

'''
	
'''
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
'''	
	
- Edit file `/etc/glance/glance-registry.conf`

	# cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.orig 
	# vi /etc/glance/glance-registry.conf 
	
'''
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

'''

- Đăng ký vào Image service database: 

	# su -s /bin/sh -c "glance-manage db_sync" glance

- Restart Image service 

	# service glance-registry restart
	# service glance-api restart

- Test 

Sử dụng CirrOS, một distro nhỏ nhẹ cho việc test Image service 

	$ . admin-openrc
	$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

- Upload image để Image service sử dụng định dạng disk là QCOW2, bare container và public để tất cả các project có thể truy cập nó: 

	$ openstack image create "cirros" \
		--file cirros-0.3.5-x86_64-disk.img \
		--disk-format qcow2 --container-format bare \
		--public

		
'''
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
'''


- Liệt kê các image 

	$ openstack image list

'''
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 0fc50da8-b841-4ddd-a457-49a639daa7a0 | cirros | active |
+--------------------------------------+--------+--------+
'''


## Compute service

Sử dụng Compute để quản lý hệ thống cloud computing. OpenStack là một phần chính của IaaS. Các module chính được implement bằng python. 

OpenStack Compute tương tác với OpenStack Identity cho việc xác thực; với OpenStack Image cho các disk, image; với Dashboard cho interface user và admin. 

// TODO bổ sung thêm 

OpenStack Compute bao gồm các thành phần :

- * nova-api service * : 
- * nova-api-metadata service * :
- * nova-compute service * : 
- * nova-placement-api service * : 
- * nova-scheduler service * :
- * nova-conductor module * : 
- * nova-cert module * : 
- * nova-consoleauth daemon * : 
- * nova-novncproxy daemon * : 
- * nova-spicehtml5proxy daemon * :
- * nova-xvpvncproxy daemon * :
- * The queue * : 
- * SQL database * :


### Cài đặt và cấu hình trên node Controller

Phần này sẽ mô tả cách cài đặt và cấu hình Compute service (Nova) trên node Controller

- Tạo các database : 
		
	# mysql

'''
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

'''

- Sử dụng file `admin-openrc`

	$ . admin-openrc
	
- Tạo user nova 

	$ openstack user create --domain default --password-prompt nova

'''
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

'''	
	
- Thêm admin role 

	$ openstack role add --project service --user nova admin
	
- Tạo nova service entity: 

	$ openstack service create --name nova \
	  --description "OpenStack Compute" compute
	  
'''
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 116ee53a5ec84eb79edc6b3b8c0a1058 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
'''


- Tạo Compute API service endpoint

	$ openstack endpoint create --region RegionOne \
	  compute public http://controller:8774/v2.1

'''
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
'''

	$ openstack endpoint create --region RegionOne \
	  compute internal http://controller:8774/v2.1

'''
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

'''

	$ openstack endpoint create --region RegionOne \
	  compute admin http://controller:8774/v2.1

'''
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
'''
	  
- Tạo user Placement

	$ openstack user create --domain default --password-prompt placement
	  
'''
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

'''

- Tạo Placement API entry trong danh mục dịch vụ

	$ openstack service create --name placement --description "Placement API" placement

'''	
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | c882480e61a74244a648fa8dd6ee2639 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
'''

- Tạo Placement API endpoint

	$ openstack endpoint create --region RegionOne placement public http://controller:8778

'''
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
'''

	$ openstack endpoint create --region RegionOne placement internal http://controller:8778

'''
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
'''

	$ openstack endpoint create --region RegionOne placement admin http://controller:8778
	
'''
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
'''	
	
- Cài đặt các packages 

	# apt install nova-api nova-conductor nova-consoleauth \
	  nova-novncproxy nova-scheduler nova-placement-api

- Edit file `/etc/nova/nova.conf`

	# cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
	# vi /etc/nova/nova.conf
	
'''
[api_database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api 		# line 3379
[database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova 			# line 4396

[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller				# line 3021

[api]
# ...
auth_strategy = keystone												# line 3085

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
my_ip = 10.0.10.10 														# line 1481

use_neutron = True														# line 2306 
firewall_driver = nova.virt.firewall.NoopFirewallDriver					# line 2465

[vnc]
enabled = true															# line 9704
# ...
vncserver_listen = $my_ip												# line 9727			
vncserver_proxyclient_address = $my_ip									# line 9739

[glance]
# ...
api_servers = http://controller:9292									# line 4953

[placement]
# ...
os_region_name = RegionOne												# line 8129
project_domain_name = Default											# line 8179
project_name = service													# line 8173
auth_type = password													# line 8155
user_domain_name = Default												# line 8205
auth_url = http://controller:35357/v3									# line 8161
username = placement													# line 8199
password = locvx1234													# line 8208
'''


- Đăng ký nove-api database
	# su -s /bin/sh -c "nova-manage api_db sync" nova

- Đăng ký cell0 database

	# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
	
- Tạo `cell1`	

	# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
	699fb0f3-e165-4ec9-9798-fe5925731a3a
	
- Đăng ký nova database

	# su -s /bin/sh -c "nova-manage db sync" nova

- Liệt kê 2 cell đã đăng ký 

	# nova-manage cell_v2 list_cells
	
'''
+-------+--------------------------------------+
|  Name |                 UUID                 |
+-------+--------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |
| cell1 | 699fb0f3-e165-4ec9-9798-fe5925731a3a |
+-------+--------------------------------------+
'''

- Khởi động lại các service

'''
# service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart


### Cài đặt và cấu hình một node Compute 

Phần này sẽ mô tả các bước cài đặt và cấu hình Compute service trên node Compute. Service này hỗ trợ một vài hyvervisor để deloy các instance hoặc VM. Để đơn giản, cấu hình này sử dụng QEMU hypervisor với KVM extension trên node compute hỗ trợ tăng tốc phần cứng cho các máy ảo. 

- Cài đặt package nova-compute 

	# apt install nova-compute
	
- Edit file `/etc/nova/nova.conf`

	# cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
	# vi /etc/nova/nova.conf
	
'''
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
password = NOVA_PASS


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
password = PLACEMENT_PASS


'''
	
Xác định xem node Compute có tăng tốc độ phần cứng cho máy ảo hay không 

	$ egrep -c '(vmx|svm)' /proc/cpuinfo
	
Nếu lệnh trả về 1 hoặc lớn hơn, node Compute hỗ trợ tăng tốc mà không cần phải cấu hình thêm

Nếu lệnh trả về 0 thì phải cấu hình libvirt để sử dụng QEMU thay vì KVM 

	# vi /etc/nova/nova-compute.conf
	
'''
[libvirt]
# ...
virt_type = qemu
'''
	
- Restart Compute service 

	# service nova-compute restart
	
### Thêm node Compute vào cell database

* Node * : Các lệnh thực hiện trên node Controller 

	$ . admin-openrc
	$ openstack hypervisor list

'''	
+----+---------------------+-----------------+--------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP      | State |
+----+---------------------+-----------------+--------------+-------+
|  1 | com1                | QEMU            | 10.10.10.100 | up    |
+----+---------------------+-----------------+--------------+-------+
'''

	# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

'''
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': 699fb0f3-e165-4ec9-9798-fe5925731a3a
Found 1 computes in cell: 699fb0f3-e165-4ec9-9798-fe5925731a3a
Checking host mapping for compute host 'com1': f00ad79a-8e68-476c-a237-25e743648a43
Creating host mapping for compute host 'com1': f00ad79a-8e68-476c-a237-25e743648a43
'''

	# vi /etc/nova/nova.conf
		
	[scheduler]
	discover_hosts_in_cells_interval = 300							# line 8703
	
### Verify operation 

* Node * : Các lệnh thực hiện trên node Controller 

	$ . admin-openrc
	$ openstack compute service list

'''
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  3 | nova-consoleauth | controller | internal | enabled | up    | 2017-07-03T08:21:35.000000 |
|  4 | nova-scheduler   | controller | internal | enabled | up    | 2017-07-03T08:21:38.000000 |
|  5 | nova-conductor   | controller | internal | enabled | up    | 2017-07-03T08:21:43.000000 |
|  6 | nova-compute     | com1       | nova     | enabled | up    | 2017-07-03T08:21:37.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
'''	
	
	$ openstack catalog list
	
	
'''
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
'''

	$ openstack image list

	
'''
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 0fc50da8-b841-4ddd-a457-49a639daa7a0 | cirros | active |
+--------------------------------------+--------+--------+
'''


	# nova-status upgrade check
	
'''

'''

## Networking service
'''
## Dashboard

## Block Storage service

## Additional service

## Launch an instance 





Tham khảo : 

https://docs.openstack.org/ocata/install-guide-ubuntu/InstallGuide.pdf