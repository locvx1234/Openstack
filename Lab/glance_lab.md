## 1. Các thao tác cơ bản 

Các image bao gồm các status : 

- Create: tạo một image sẵn trước để có thể upload lên glance
- Queued: Image đang ở trong hàng đợi (hàng đợi chỉ dành cho image). Registry Glance sẽ đảm nhận chức năng nhận diện image. Ở trạng thái này, dữ liệu của image chưa được tải lên Glance
- Saving: Dữ liệu của image đang được upload lên Glance
- Active: Dữ liệu của image đã hoàn toàn được tải lên Glance
- Deactivated: Image không được phép truy cập vào, kể cả admin
- Killed: Có lỗi xảy ra trong quá trình truyền tải dữ liệu của image
- Deleted: Glance dữ lại các thông tin về image nhưng image không còn được sử dụng. Image này sẽ được xóa đi vào một ngày sau đó

![statusflow](https://github.com/locvx1234/Openstack/blob/master/images/statusflow.jpg)


Trước khi làm việc với Keystone, cần khai báo các biến môi trường để xác thực.

$ . admin-openrc  

### Liệt kê các image 
```
root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | active |
+--------------------------------------+--------+--------+
```


### Xem thông tin chi tiết về image 

```
root@controller:~# openstack image show a505feb5-3f25-4e3c-acb1-1f3d35e27747
```
or
```
root@controller:~# openstack image show cirris
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2017-08-15T03:34:53Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/a505feb5-3f25-4e3c-acb1-1f3d35e27747/file |
| id               | a505feb5-3f25-4e3c-acb1-1f3d35e27747                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | 1e1f11c83cf24288b10524f3a05878e6                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-08-15T03:34:54Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

### Kiểm tra thông tin image trong database

```
MariaDB [(none)]> select * from glance.image_locations\G
*************************** 1. row ***************************
        id: 1
  image_id: a505feb5-3f25-4e3c-acb1-1f3d35e27747
     value: file:///var/lib/glance/images/a505feb5-3f25-4e3c-acb1-1f3d35e27747
created_at: 2017-08-15 03:34:54
updated_at: 2017-08-15 03:34:54
deleted_at: NULL
   deleted: 0
 meta_data: {}
    status: active
1 row in set (0.00 sec)

MariaDB [(none)]>
```

### Deactivated image

```
root@controller:~# openstack image set --deactivate cirros
root@controller:~# openstack image list
+--------------------------------------+--------+-------------+
| ID                                   | Name   | Status      |
+--------------------------------------+--------+-------------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | deactivated |
+--------------------------------------+--------+-------------+
```

### Reactivate image 

```
root@controller:~# openstack image set --activate cirros
root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | active |
+--------------------------------------+--------+--------+
```

### Delete image
 
```
root@controller:~# openstack image delete cirros
root@controller:~# openstack image list

root@controller:~#
```

Kiểm tra trong database

```
MariaDB [(none)]> select * from glance.image_locations\G
*************************** 1. row ***************************
        id: 1
  image_id: a505feb5-3f25-4e3c-acb1-1f3d35e27747
     value: file:///var/lib/glance/images/a505feb5-3f25-4e3c-acb1-1f3d35e27747
created_at: 2017-08-15 03:34:54
updated_at: 2017-08-28 11:13:23
deleted_at: 2017-08-28 11:13:23
   deleted: 1
 meta_data: {}
    status: deleted
1 row in set (0.00 sec)

MariaDB [(none)]>
```

### Upload image 

Tải một image có sẵn từ internet 

```
wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
```

Upload file vừa tải 
```
 root@controller:~# openstack image create "cirros-test-upload-image" \
>  --file cirros-0.3.5-x86_64-disk.img \
>  --disk-format qcow2 --container-format bare \
>  --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2017-08-28T11:17:08Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5/file |
| id               | c420b371-7cb3-41d2-abc6-a4bb56c420e5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros-test-upload-image                             |
| owner            | 1e1f11c83cf24288b10524f3a05878e6                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-08-28T11:17:10Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
 ```
 
 Xác nhận lại image được upload thành công sẽ ở trạng thái active  
 ```
 root@controller:~# openstack image list
+--------------------------------------+--------------------------+--------+
| ID                                   | Name                     | Status |
+--------------------------------------+--------------------------+--------+
| c420b371-7cb3-41d2-abc6-a4bb56c420e5 | cirros-test-upload-image | active |
+--------------------------------------+--------------------------+--------+
 ```
 
 ## 2. Lưu image ở nhiều nơi (multiple store location)

Tạo 2 thư mục lưu image và thay đổi chủ sở hữu 
```
root@controller:~#  mkdir /home/images-1
root@controller:~#  mkdir /home/images-2
root@controller:~# chown glance:glance /home/images-1
root@controller:~# chown glance:glance /home/images-2
```

Sửa file cấu hình 
```
root@controller:~# vi /etc/glance/glance-api.conf

[glance_store]
filesystem_store_datadir=
filesystem_store_datadirs=/var/lib/glance/images
filesystem_store_datadirs=/home/images-1:200
filesystem_store_datadirs=/home/images-2:100
```

Khởi động lại dịch vụ

```
service glance-api restart
```

Chỉ số sau đường dẫn là độ ưu tiên lưu trữ, mặc định là 0. Theo cấu hình trên thì image được ưu tiên lưu trữ trong /images-1  trước tiên.

Kiểm tra điều này bằng cách tạo một image mới.

```
root@controller:~# openstack image create "cirros-multiple-store-location" \
--file cirros-0.3.5-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2017-08-29T07:29:13Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/559f4246-8500-40a8-b4bd-0fb1743e0360/file |
| id               | 559f4246-8500-40a8-b4bd-0fb1743e0360                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros-multiple-store-location                       |
| owner            | 1e1f11c83cf24288b10524f3a05878e6                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-08-29T07:29:13Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

Kiểm tra vị trí lưu image 

```
MariaDB [(none)]> select * from glance.image_locations\G
*************************** 1. row ***************************
        id: 1
  image_id: a505feb5-3f25-4e3c-acb1-1f3d35e27747
     value: file:///var/lib/glance/images/a505feb5-3f25-4e3c-acb1-1f3d35e27747
created_at: 2017-08-15 03:34:54
updated_at: 2017-08-28 11:13:23
deleted_at: 2017-08-28 11:13:23
   deleted: 1
 meta_data: {}
    status: deleted
*************************** 2. row ***************************
        id: 2
  image_id: c420b371-7cb3-41d2-abc6-a4bb56c420e5
     value: file:///var/lib/glance/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5
created_at: 2017-08-28 11:17:10
updated_at: 2017-08-28 11:17:10
deleted_at: NULL
   deleted: 0
 meta_data: {}
    status: active
*************************** 3. row ***************************
        id: 3
  image_id: 559f4246-8500-40a8-b4bd-0fb1743e0360
     value: file:///home/images-1/559f4246-8500-40a8-b4bd-0fb1743e0360
created_at: 2017-08-29 07:29:13
updated_at: 2017-08-29 07:29:13
deleted_at: NULL
   deleted: 0
 meta_data: {}
    status: active
3 rows in set (0.00 sec)

MariaDB [(none)]>
```

## 3. Các lệnh với curl

Trước tiên phải có token, [xem thêm](https://github.com/locvx1234/Openstack/blob/master/Lab/keystone_lab.md)

Lấy thông tin các image 

```
curl -s -X GET -H "X-Auth-Token: $OS_TOKEN" http://10.10.10.61:9292/v2/images | python -mjson.tool
```












