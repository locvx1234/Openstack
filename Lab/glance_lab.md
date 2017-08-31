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
curl -s -X GET -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images | python -mjson.tool
{
    "first": "/v2/images",
    "images": [
        {
            "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290",
            "container_format": "bare",
            "created_at": "2017-08-29T07:29:13Z",
            "disk_format": "qcow2",
            "file": "/v2/images/559f4246-8500-40a8-b4bd-0fb1743e0360/file",
            "id": "559f4246-8500-40a8-b4bd-0fb1743e0360",
            "min_disk": 0,
            "min_ram": 0,
            "name": "cirros-multiple-store-location",
            "owner": "1e1f11c83cf24288b10524f3a05878e6",
            "protected": false,
            "schema": "/v2/schemas/image",
            "self": "/v2/images/559f4246-8500-40a8-b4bd-0fb1743e0360",
            "size": 13267968,
            "status": "active",
            "tags": [],
            "updated_at": "2017-08-29T07:29:13Z",
            "virtual_size": null,
            "visibility": "public"
        },
        {
            "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290",
            "container_format": "bare",
            "created_at": "2017-08-28T11:17:08Z",
            "disk_format": "qcow2",
            "file": "/v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5/file",
            "id": "c420b371-7cb3-41d2-abc6-a4bb56c420e5",
            "min_disk": 0,
            "min_ram": 0,
            "name": "cirros-test-upload-image",
            "owner": "1e1f11c83cf24288b10524f3a05878e6",
            "protected": false,
            "schema": "/v2/schemas/image",
            "self": "/v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5",
            "size": 13267968,
            "status": "active",
            "tags": [],
            "updated_at": "2017-08-28T11:17:10Z",
            "virtual_size": null,
            "visibility": "public"
        }
    ],
    "schema": "/v2/schemas/images"
}
```

Xem chi tiết về image `c420b371-7cb3-41d2-abc6-a4bb56c420e5`

```
curl -s -X GET -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5 | python -mjson.tool
{
    "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290",
    "container_format": "bare",
    "created_at": "2017-08-28T11:17:08Z",
    "disk_format": "qcow2",
    "file": "/v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5/file",
    "id": "c420b371-7cb3-41d2-abc6-a4bb56c420e5",
    "min_disk": 0,
    "min_ram": 0,
    "name": "cirros-test-upload-image",
    "owner": "1e1f11c83cf24288b10524f3a05878e6",
    "protected": false,
    "schema": "/v2/schemas/image",
    "self": "/v2/images/c420b371-7cb3-41d2-abc6-a4bb56c420e5",
    "size": 13267968,
    "status": "active",
    "tags": [],
    "updated_at": "2017-08-28T11:17:10Z",
    "virtual_size": null,
    "visibility": "public"
}
```

Tạo một image chưa có dữ liệu
```
curl -i -X POST -H "X-Auth-Token: $OS_TOKEN" \
-H "Content-Type: application/json" \
-d '{"container_format": "bare", "disk_format": "qcow2", "name": "cirros-test-curl"}' \
http://controller:9292/v2/images 
```
Kết quả
```
HTTP/1.1 201 Created
Content-Length: 560
Content-Type: application/json; charset=UTF-8
Location: http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200
X-Openstack-Request-Id: req-da34595d-293a-464c-aef3-21bfc2f924e0
Date: Thu, 31 Aug 2017 09:02:05 GMT

{"status": "queued", 
"name": "cirros-test-curl", 
"tags": [], 
"container_format": "bare", 
"created_at": "2017-08-31T09:02:05Z", 
"size": null, 
"disk_format": "qcow2", 
"updated_at": "2017-08-31T09:02:05Z", 
"visibility": "shared", 
"self": "/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200", 
"min_disk": 0, "protected": false, 
"id": "e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200", 
"file": "/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200/file", 
"checksum": null, 
"owner": "1e1f11c83cf24288b10524f3a05878e6", 
"virtual_size": null, 
"min_ram": 0, 
"schema": "/v2/schemas/image"}
```

Status của image là queued cho biết image đang sẵn sàng để upload dữ liệu. 

Upload dữ liệu cho image vừa tạo 
```
curl -i -X PUT -H "X-Auth-Token: $OS_TOKEN" \
-H "Content-Type: application/octet-stream" \
-d @/root/cirros-0.3.5-x86_64-disk.img \
http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200/file
```

Kết quả :
```
HTTP/1.1 100 Continue

HTTP/1.1 204 No Content
Content-Type: text/html; charset=UTF-8
Content-Length: 0
X-Openstack-Request-Id: req-4b856c4f-8fd4-404b-819c-494f72719c47
Date: Thu, 31 Aug 2017 09:34:48 GMT
```

Status image lúc này sẽ là active
```
root@controller:~# curl -s -X GET -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200 | python -mjson.tool
{
    "checksum": "5330265d2649c25ba62260b7e4640474",
    "container_format": "bare",
    "created_at": "2017-08-31T09:02:05Z",
    "disk_format": "qcow2",
    "file": "/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200/file",
    "id": "e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200",
    "min_disk": 0,
    "min_ram": 0,
    "name": "cirros-test-curl",
    "owner": "1e1f11c83cf24288b10524f3a05878e6",
    "protected": false,
    "schema": "/v2/schemas/image",
    "self": "/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200",
    "size": 6893708,
    "status": "active",
    "tags": [],
    "updated_at": "2017-08-31T09:34:48Z",
    "virtual_size": null,
    "visibility": "shared"
}
```

Deactivate image
```
root@controller:~# curl -i -X POST -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200/actions/deactivate
HTTP/1.1 204 No Content
Content-Type: text/html; charset=UTF-8
Content-Length: 0
X-Openstack-Request-Id: req-15fea8c4-0f36-4690-af4c-c2b43f5ece13
Date: Thu, 31 Aug 2017 09:38:13 GMT
```

Reactivate image 

```
root@controller:~# curl -i -X POST -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200/actions/reactivate
HTTP/1.1 204 No Content
Content-Type: text/html; charset=UTF-8
Content-Length: 0
X-Openstack-Request-Id: req-73970542-b8e5-4b34-b17c-df95c393c04d
Date: Thu, 31 Aug 2017 09:38:59 GMT
```

Delete image 

```
root@controller:~# curl -i -X DELETE -H "X-Auth-Token: $OS_TOKEN" http://controller:9292/v2/images/e8d2fe2f-1791-4bcd-b12e-7abcdb6d7200
HTTP/1.1 204 No Content
Content-Type: text/html; charset=UTF-8
Content-Length: 0
X-Openstack-Request-Id: req-4ac51881-2214-4fe8-b34e-a7e37945fdd7
Date: Thu, 31 Aug 2017 09:39:43 GMT
```






