DevStack là một bộ các script giúp nhanh chóng cài đặt một môi trường OpenStack cơ bản. Nó được sử dụng để làm môi trường test rất tiện lợi.

**Warning** : DevStack sẽ thay đổi nhiều thứ trong quá trình cài đặt. Do đó bạn cần sử dụng một server hoặc VM riêng cho việc này.

## Quick start 

### Install Linux 

DevStack hỗ trợ trên nhiều distro như Ubuntu 16.04/17.04, Fedora 24/25, CentOS/RHEL 7, Debian và OpenSUSE. Trong đó Ubuntu 16.04 được ưa chuộng nhất.

### Thêm user `stack`

```sh
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo su - stack
```

### Download DevStack

**Note** : Cần cài đặt git trước nếu chưa có 

```sh
$ git clone https://git.openstack.org/openstack-dev/devstack
$ cd devstack
```

### Create a local.conf

Tạo một file `local.conf` với 4 password tương ứng 

```
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```

### Start the install

```sh
./stack.sh
```

Quá trình này sẽ mất 15-20 phút tùy thuộc vào tốc độ mạng.

### Profit!

Khi DevStack được cài thành công, DevStack sẽ được cài đặt `keystone`, `glance`, `nova`, `cinder`, `neutron` và `horizon`. 

Bạn có thể truy cập vào horizon để trải nghiệm giao diện web của OpenStack và quản lý các VM, network, volume và image.

Bạn cũng có thể sử dụng lệnh `openstack` trên giao diện shell để quản lý devstack.



