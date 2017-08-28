Các image bao gồm các status : 

- Create: tạo một image sẵn trước để có thể upload lên glance
- Queued: Image đang ở trong hàng đợi (hàng đợi chỉ dành cho image). Registry Glance sẽ đảm nhận chức năng nhận diện image. Ở trạng thái này, dữ liệu của image chưa được tải lên Glance
- Saving: Dữ liệu của image đang được upload lên Glance
- Active: Dữ liệu của image đã hoàn toàn được tải lên Glance
- Deactivated: Image không được phép truy cập vào, kể cả admin
- Killed: Có lỗi xảy ra trong quá trình truyền tải dữ liệu của image
- Deleted: Glance dữ lại các thông tin về image nhưng image không còn được sử dụng. Image này sẽ được xóa đi vào một ngày sau đó



Trước khi làm việc với Keystone, cần khai báo các biến môi trường để xác thực.

$ . admin-openrc  

###Liệt kê các image 
```
root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | active |
+--------------------------------------+--------+--------+
```


###Xem thông tin chi tiết về image 

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

###Kiểm tra thông tin image trong database

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

###Deactivated image

```
root@controller:~# openstack image set --deactivate cirros
root@controller:~# openstack image list
+--------------------------------------+--------+-------------+
| ID                                   | Name   | Status      |
+--------------------------------------+--------+-------------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | deactivated |
+--------------------------------------+--------+-------------+
```

###Reactivate image 

```
root@controller:~# openstack image set --activate cirros
root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| a505feb5-3f25-4e3c-acb1-1f3d35e27747 | cirros | active |
+--------------------------------------+--------+--------+
```

###Delete image
 
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

###Upload image 

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
 
 



