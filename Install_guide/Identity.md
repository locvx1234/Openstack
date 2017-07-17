:arrow_left: [Environment](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Environment.md)

## Identity service (Keystone)

Identity service quản lý việc xác thực, cấp phép và một service catalog.

Identity service thường là service đầu tiên mà user tương tác. Một khi được xác thực, end user có thể sử dụng định danh của họ để truy cập các service khác của OpenStack.

Identity service cũng có thể tích hợp với một số hệ thống quản lý người dùng (như LDAP)

Các user và service có thể sử dụng các service catalog được quản lý bởi Identity service. Mỗi service có 1 hoặc nhiều endpoint thuộc 1 trong 3 loại : admin, internal hoặc public

Ví dụ : 

Public API network  có thể nhìn thấy từ Internet để khách hàng có thể quản lý các cloud của họ. 

Admin API network có thể bị giới hạn hoạt động trong các tổ chức quản lý hạ tầng cloud

Internal API network có thể bị giới hạn đối với máy chủ có chứa các dịch vụ OpenStack

Identity service chứa 3 thành phần : 

- **Server** : Một server tập trung cung cấp dịch vụ authentication và authorization sử dụng một RESTful interface
- **Drivers** : Các drive hoặc một service back end được tích hợp vào server tập trung. Chúng sử dụng để truy cập thông tin trong các repo bên ngoài OpenStack và có thể tồn tại trong hạ tầng mà OpenStack deploy (ví dụ SQL database, LDAP server)
- **Modules** :  Các module middleware trong OpenStack sử dụng Identity service

### Cài đặt và cấu hình 

Phần này mô tả các bước cài đặt và cấu hình Identity service - Keystone trên Controller. Cho mục đích mở rộng, cấu hình này deploy Fernet token và Apache HTTP server để xử lý yêu cầu.

Truy cập cơ sở dữ liệu: 

	# mysql -u root -p
	
Tạo keystone database:
	
	MariaDB [(none)]> CREATE DATABASE keystone default character set utf8;

Cấp quyền truy cập cho keystone database với KEYSTONE_DBPASS là `locvx1234` :
	
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'locvx1234' WITH GRANT OPTION;
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'locvx1234' WITH GRANT OPTION;
	MariaDB [(none)]> FLUSH PRIVILEGES;
	
Exit database access 

	MariaDB [(none)]> exit
	
**Note**: Apache HTTP server với `mod_wsgi` để đáp ứng các request Identity service trên cổng 5000 và 35357 

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

```sh
$ export OS_USERNAME=admin
$ export OS_PASSWORD=locvx1234
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
	
```

### Tạo domain, projects, users và các role

```
$ openstack project create --domain default \
--description "Service Project" service
```

```
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
```

Những task thông thường (không phải admin) nên sử dụng một project và user không có nhiều quyền. Ví dụ :

- Tạo project 

```
$ openstack project create --domain default \
--description "Demo Project" demo	
```
	
```
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
```
	
- Tạo user 

```
$ openstack user create --domain default \
--password-prompt demo
```	
	
```
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
```
	
- Tạo user role 

```
$ openstack role create user
```

```
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 9ec4783f76504dd89fbb816e7438e54e |
| name      | user                             |
+-----------+----------------------------------+
```

Thêm user role vào project và user demo 

```
$ openstack role add --project demo --user demo user
```

### Xác nhận hoạt động 

Cần verify lại Identity service trước khi cài đặt các service khác. 

**Note** : Các lệnh thực hiện trên node Controller  	
	
Vì lý do bảo mật, disable cơ chế xác thực token tạm thời 

```
# cp /etc/keystone/keystone-paste.ini /etc/keystone/keystone-paste.ini.orig
# vi /etc/keystone/keystone-paste.ini
```

Xóa `admin_token_auth` trong các phần [pipeline:public_api], [pipeline:admin_api], và [pipeline:api_v3]

Unset biến môi trường y OS_AUTH_URL và OS_PASSWORD

```
$ unset OS_AUTH_URL OS_PASSWORD
```

Request một xác thực token với user admin : 

```
$ openstack --os-auth-url http://controller:35357/v3 \
--os-project-domain-name default --os-user-domain-name default \
--os-project-name admin --os-username admin token issue
```

```
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-06-30T06:10:30+0000                                                                                                     |
| id         | gAAAAABZVd1GcSwhPb0TjXlV9Nq012GyeORRttY2Q0IssbWCQ6Lsuu8aIT1BAJLnjg6hVxy4ZxsSscwGZ8tJBoy95Q9q7aIoIq6WO9qAcdGEcMTJkoHzZRw6SPNu |
|            | _IjMlNGe23ev7heNcIv_RdXzUY-PMbz1LE4CePhNCC2vAdPBkqRG45BgXXI                                                                  |
| project_id | 210387a5dc824ac2aaa93fdee42784ed                                                                                             |
| user_id    | ee69e3d27c3c4e25b2e363b79f408e2f                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+
```


Request một xác thực token với user demo : 

```
$ openstack --os-auth-url http://controller:5000/v3 \
--os-project-domain-name default --os-user-domain-name default \
--os-project-name demo --os-username demo token issue
```

```
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-06-30T06:12:03+0000                                                                                                     |
| id         | gAAAAABZVd2j7lJVeIPp1r2ZdMf38KC1kQHBiFqdH-_dCKN0RviP9fGsDeVLOdBwIRnMPmbIDQY5Ym4ZSlu1XCe9wdSSaoLRc4n8G_tMa9Ayz6_ldxau8DXL_UcQ |
|            | e90rXFCoYvaUHgAbDik1OdDZvA1GrYyPdSNtjQfjwgz73L8DxM6jBDwuII8                                                                  |
| project_id | b167aaf8ce33471d9683988868bbd048                                                                                             |
| user_id    | 75a4a173328b467a81dc6e3fe56af74b                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+
```

### Script 

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

File `demo-openrc`:

```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=locvx1234
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
	
Sử dụng script : 

```
$ . admin-openrc
$ openstack token issue
```

:arrow_right: [Image service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Image.md)