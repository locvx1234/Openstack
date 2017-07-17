:arrow_left: [Content](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Install_guide.md)

## Overview 

Openstack là một nền tảng điện toán đám mây với mục đích đơn giản hóa việc thực hiện, khả năng mở rộng cao và tính năng phong phú. 

Tính đến thời điểm hiện tại là bản Release OpenStack thứ 15, có tới 1925 developer trên toàn thế giới tham gia đóng góp cho dự án này.

**Note**: Bài lab này sử dụng để tìm hiểu về OpenStack. Mô hình product sẽ phức tạp hơn

Sau khi làm quen với cài đặt cơ bản, cấu hình, xử lý sự cố các dịch vụ của OpenStack, các bước để triển khai product : 

- Xác định và implement các dịch vụ cốt lõi và các tùy chọn cần thiết để đáp ứng hiệu năng và dự phòng

- Tăng cường an ninh bằng các phương pháp như firewall, mã hóa, các policy dịch vụ 

- Sử dụng các công cụ như Ansible, Chef, Puppet, Salt để tự động triển khai và quản lý product.

<a name="example_architecture">
### Example architecture 

Mô hình lab này cần tối thiểu 2 node (host) để chạy một virtual machine hoặc instance.

Các service tùy chọn như Block Storage và Object Storage thì cần thêm các node bổ sung. 

Mô hình này là mô hình có cấu hình tối thiểu khác với mô hình triển khai product : 

- Các Networking agent được cài ngay trên Node Controller thay vì trên một node Network chuyên biệt

- Network dành cho trao đổi lưu lượng giữa các máy trong self-service networks sử dụng management network thay vì tách thành một đường mạng riêng 
 
Yêu cầu phần cứng :

![Hardware requirement](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/hardware%20requeirement.png) 

- *Giải thích các thành phần*

#### Controller

Node Controller chạy Identity service, Image service, quản lý Compute, quản lý Networking, Networking agent và Dashboard. Nó cũng bao gồm hỗ trợ các dịch vụ như SQL database, message queue, NTP

Tùy chọn, Node Controller có thể chạy Block Storage, Object Storage, Orchestration, Telemetry. 

Node Controller cần ít nhất 2 network interface.

#### Compute 

Node Compute chạy phần hypervisor của Compute để điều khiển instance (VM). Mặc định, Compute sử dụng KVM hypervisor. 

Node Compute cũng chạy một agent Networking service, nó sẽ connect các instance để ảo hóa network và cung cấp firewall cho các service.

Có thể deploy nhiều hơn 1 node Compute. Mỗi node tối thiểu 2 network interface.


#### Block Storage

Node tùy chọn: Block Storage chứa các disk sử dụng cho các dịch vụ Block Storage và Shared File System để cũng cấp cho instance.

Để đơn giản, traffic của service giữa các node compute và node này sử dụng chung với đường management. Còn trên môi trường product, nên triển khai một đường mạng riêng để tăng hiệu quả và an ninh.

Có thể deploy nhiều hơn 1 node Block Storage. Mỗi node tối thiểu 2 network interface.


#### Object Storage

Node tùy chọn : Object Storage chứa các disk sử dụng cho các dịch vụ Object Storage để sử dụng cho việc lưu trữ các account, container và object.

Để đơn giản, traffic của service giữa các node Compute và node này sử dụng chung với đường management. Còn trên môi trường product, nên triển khai một đường mạng riêng để tăng hiệu quả và an ninh.

Có thể deploy nhiều hơn 1 node Block Storage. Mỗi node tối thiểu 2 network interface.

Service này cần 2 node. Mỗi node tối thiểu 2 network interface. Có thể deploy nhiều hơn 2 node.

#### Networking

Có 2 mô hình 

##### Networking option 1: Provider network

Các service chủ yếu là layer 2 (bridging/switching) và VLAN. Về cơ bản, nó làm cầu nối các mạng ảo với các mạng vật lý trên cơ sở hạ tầng mạng vật lý chơ các service layer 3.

**Warning**: Option này không hỗ trợ cho mạng self-service (private), layer 3 (routing) service, và các service nâng cao như LBaaS, FWaaS.

![provider](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/networking_opt1.png)
##### Networking option 2: Self-service network

Mô hình này cung cấp các service layer 3. Về cơ bản, nó sẽ định tuyến các mạng ảo tới các mạng vật lý sử dụng NAT. Ngoài ra, tùy chọn này cung cấp nền tảng cho các dịch vụ như LBaaS, FWaaS.

![self-service](https://raw.githubusercontent.com/locvx1234/Openstack/master/images/networking_opt2.png)


:arrow_right: [Environment](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Environment.md)
