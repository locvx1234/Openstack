:arrow_left: [Overview](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Overview.md)

## Environment

Phần này giải thích cách cấu hình node Controller và node Compute sử dụng [Example architecture](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Overview.md#example_architecture).

Hầu hết các môi trường bao gồm Identity service , Image service, Compute, Networking còn Dashboard, Block Storage, Object Storage và các thành phần khác là tùy chọn.

Yêu cầu tối thiểu để sử dụng với những OS nhỏ nhẹ như CirrOS:

- Controller Node: 1 processor, 4 GB memory, 5 GB storage
- Compute Node: 1 processor, 2 GB memory, 10 GB storage

Nếu hiệu suất giảm khi enable thêm các service hoặc máy ảo, khi đó hãy xem xét bổ sung phần cứng.

Một phân vùng disk trên mỗi node được sử dụng cho các cài đặt cơ bản. Tuy nhiên, bạn nên xem xét LVM cho việc cài các service tùy chọn như Block Storage.

Cho lần cài đầu tiên, nhiều người sẽ chọn cách cài trên máy ảo vì những ưu điểm của nó, nhưng nó cũng làm giảm hiệu suất của instance.

### Security 

Các service OpenStack  hỗ trợ service theo các phương thức như password, policy, encryption. 

Để đơn giản quá trình cài đặt, hướng dẫn này chỉ bảo mật theo password.

Password có thể tạo bằng tay, nhưng chuỗi kết nối database trong file cấu hình không được chứa ký tự đặc biệt như "@"/

Recommend sử dụng [pwgen](https://sourceforge.net/projects/pwgen/) hoặc chạy lệnh sau: 

	$ openssl rand -hex 10

Trong guide này sử dụng SERVICE_PASS để sử dụng cho password các service, SERVICE_DBPASS sử dụng cho password của database.

Đây là các biến pass và mình thay thế tất cả đều là `locvx1234` cho dễ nhớ :)) 

![Password](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/password.png)


### Host networking

Sau khi cài đặt OS trên mỗi node, bạn phải cấu hình network interface. 

![layer](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/layout.png)

![ip_planning](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/ip_planning.png)

### Network Time Protocol (NTP)

Chúng ta cần phải đồng bộ thời gian cho các node. Để cài đặt NTP, ta sử dụng Chrony 

#### Trên Controller 

Cài đặt gói chrony cho NTP server

	# apt install -y chrony
Sao lưu file cấu hình của NTP Server

	# cp /etc/chrony/chrony.conf /etc/chrony/chrony.conf.orig
Cấu hình NTP server

```
echo "server 1.vn.pool.ntp.org iburst 
server 0.asia.pool.ntp.org iburst 
server 3.asia.pool.ntp.org iburst

allow 10.10.10.0/24" >> /etc/chrony/chrony.conf
```

Khởi động lại dịch vụ NTP trên Controller

	# systemctl restart chrony
Kiểm tra dịch vụ NTP đã hoạt động hay chưa

	# chronyc sources

Output :

```
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
```	
	
Tất cả các node khác tham khảo node Controller để đồng bộ đồng hồ. Thực hiên tương tự trên tất cả các node khác.

Thay vì echo các server ntp.org thì comment dòng 

```
#pool 2.debian.pool.ntp.org offline iburst
```
và thêm 
```
server controller iburst
```

Sau đó restart lại dịch vụ và kiểm tra

```
root@compute:~# service chrony restart
root@compute:~# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? controller                    0   6     0   10y     +0ns[   +0ns] +/-    0ns
```


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

Các node khác cài đặt tương tự 
	
### SQL database

Hầu hết các dịch vụ OpenStack đều cần SQL database để lưu trữ thông tin. Database thường chạy trên node Controller.

Chúng ta sẽ sử dụng MariaDB hoặc MySQL trong distro này, bên cạnh đó, các service OpenStack cũng hỗ trợ tốt với các SQL database khác như PostgreSQL.

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

	# apt install rabbitmq-server -y
	
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
	
	sed -i 's/-l 127.0.0.1/-l 0.0.0.0/g' /etc/memcached.conf
	
Khởi động lại memcached

	systemctl enable memcached.service
	systemctl start memcached.service
	
	
:arrow_right: [Identity service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Identity.md)