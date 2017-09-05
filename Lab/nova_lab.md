## Flavor

Flavor dùng để định nghĩa tài nguyên để cung cấp cho instance (RAM, disk, vCPU,...)

### Liệt kê flavor
```
root@controller:~# . admin-openrc
root@controller:~# openstack flavor list
+----+---------+-----+------+-----------+-------+-----------+
| ID | Name    | RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+---------+-----+------+-----------+-------+-----------+
| 0  | m1.nano |  64 |    1 |         0 |     1 | True      |
+----+---------+-----+------+-----------+-------+-----------+
```

### Xem chi tiết flavor
```
root@controller:~# openstack flavor show m1.nano
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| access_project_ids         | None    |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```
### Tạo flavor

Mặc định flavor khi tạo ra là public, nghĩa là nó có thể được các project 

```
root@controller:~# openstack flavor create --id auto --ram 512 --disk 5 --vcpu 1 --public f_public.small
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 5                                    |
| id                         | ed8a0257-f4ca-43f8-a45f-c167ca16c7a0 |
| name                       | f_public.small                       |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 512                                  |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

### Tạo flavor private
```
root@controller:~# openstack flavor create --id auto --ram 512 --disk 5 --vcpu 1 --private f_private.small
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 5                                    |
| id                         | 3eaf942e-2f95-4d8e-8e88-4d7253c29a70 |
| name                       | f_private.small                      |
| os-flavor-access:is_public | False                                |
| properties                 |                                      |
| ram                        | 512                                  |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

### Liệt kê flavor 

```
root@controller:~# openstack flavor list --all
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+
| ID                                   | Name            | RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+
| 0                                    | m1.nano         |  64 |    1 |         0 |     1 | True      |
| 3eaf942e-2f95-4d8e-8e88-4d7253c29a70 | f_private.small | 512 |    5 |         0 |     1 | False     |
| ed8a0257-f4ca-43f8-a45f-c167ca16c7a0 | f_public.small  | 512 |    5 |         0 |     1 | True      |
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+

```

### Xóa flavor 
```
root@controller:~# openstack flavor delete f_public.small
root@controller:~# openstack flavor list --all
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+
| ID                                   | Name            | RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+
| 0                                    | m1.nano         |  64 |    1 |         0 |     1 | True      |
| 3eaf942e-2f95-4d8e-8e88-4d7253c29a70 | f_private.small | 512 |    5 |         0 |     1 | False     |
+--------------------------------------+-----------------+-----+------+-----------+-------+-----------+
```

## Keypair 

Tương tự sử dụng key trong ssh, keypair cũng sử dụng để đăng nhập vào instance mà không cần mật khẩu.


### Tạo key 
```
root@controller:~# ssh-keygen -q -N ""
Enter file in which to save the key (/root/.ssh/id_rsa): mykey
root@controller:~# ls
admin-openrc  cirros-0.3.5-x86_64-disk.img  demo-openrc  mykey  mykey.pub
```

### Add key vào openstack 
```
root@controller:~# openstack keypair create --public-key ~/mykey.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | d5:51:c8:a0:c2:d7:4c:29:69:f1:cf:b1:36:18:6e:05 |
| name        | mykey                                           |
| user_id     | 9c57a9c095ad42d1be6448e1452af154                |
+-------------+-------------------------------------------------+
```

### Liệt kê key 
```
root@controller:~# openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | d5:51:c8:a0:c2:d7:4c:29:69:f1:cf:b1:36:18:6e:05 |
+-------+-------------------------------------------------+
```

## VM

### Liệt kê network dể lấy ID 

```
+--------------------------------------+-------------+--------------------------------------+
| ID                                   | Name        | Subnets                              |
+--------------------------------------+-------------+--------------------------------------+
| 16015672-842c-48f3-bea5-bd077b046dd1 | provider    | 214716cd-42c1-419e-9e9b-9b1e2a25f118 |
| 611fc2cc-dcf6-4f67-bb30-8f0518dc2fd6 | selfservice | 6c74e4eb-f394-418f-bd35-0f2c56851b99 |
+--------------------------------------+-------------+--------------------------------------+
```

### Tạo máy ảo với ID của selfsive netowrk
```
root@controller:~# openstack server create --flavor m1.nano --image cirros-test-upload-image \
--nic net-id=611fc2cc-dcf6-4f67-bb30-8f0518dc2fd6 --security-group default --key-name mykey VM-test-key
+-----------------------------+-----------------------------------------------------------------+
| Field                       | Value                                                           |
+-----------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                          |
| OS-EXT-AZ:availability_zone |                                                                 |
| OS-EXT-STS:power_state      | NOSTATE                                                         |
| OS-EXT-STS:task_state       | scheduling                                                      |
| OS-EXT-STS:vm_state         | building                                                        |
| OS-SRV-USG:launched_at      | None                                                            |
| OS-SRV-USG:terminated_at    | None                                                            |
| accessIPv4                  |                                                                 |
| accessIPv6                  |                                                                 |
| addresses                   |                                                                 |
| adminPass                   | v9xPN8jKggRw                                                    |
| config_drive                |                                                                 |
| created                     | 2017-09-04T10:01:14Z                                            |
| flavor                      | m1.nano (0)                                                     |
| hostId                      |                                                                 |
| id                          | 881ff1b4-69ff-4d0c-a0ae-a4123f634b08                            |
| image                       | cirros-test-upload-image (c420b371-7cb3-41d2-abc6-a4bb56c420e5) |
| key_name                    | mykey                                                           |
| name                        | VM-test-key                                                     |
| progress                    | 0                                                               |
| project_id                  | 5a7d5cc6c723427e8d0394f70201af54                                |
| properties                  |                                                                 |
| security_groups             | name='default'                                                  |
| status                      | BUILD                                                           |
| updated                     | 2017-09-04T10:01:15Z                                            |
| user_id                     | aa638175684b41e1b0473fbe4b90bd6e                                |
| volumes_attached            |                                                                 |
+-----------------------------+-----------------------------------------------------------------+
```

### Liệt kê VM
```
root@controller:~# openstack server list
+--------------------------------------+-------------+--------+------------------------+--------------------------+
| ID                                   | Name        | Status | Networks               | Image Name               |
+--------------------------------------+-------------+--------+------------------------+--------------------------+
| 881ff1b4-69ff-4d0c-a0ae-a4123f634b08 | VM-test-key | ACTIVE | selfservice=10.10.10.5 | cirros-test-upload-image |
+--------------------------------------+-------------+--------+------------------------+--------------------------+
```

Lấy đường dẫn để console tới VM

```
root@controller:~# openstack console url show VM-test-key
+-------+---------------------------------------------------------------------------------+
| Field | Value                                                                           |
+-------+---------------------------------------------------------------------------------+
| type  | novnc                                                                           |
| url   | http://controller:6080/vnc_auto.html?token=decbdce4-1af6-417b-a1bf-bdb8483d619d |
+-------+---------------------------------------------------------------------------------+
```

### Floating IP 
```
root@controller:~# openstack floating ip create provider
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2017-09-04T10:13:41Z                 |
| description         |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 192.168.238.155                      |
| floating_network_id | 16015672-842c-48f3-bea5-bd077b046dd1 |
| id                  | cb8e8722-fca4-47db-a916-46f97090be16 |
| name                | None                                 |
| port_id             | None                                 |
| project_id          | 5a7d5cc6c723427e8d0394f70201af54     |
| revision_number     | 1                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| updated_at          | 2017-09-04T10:13:41Z                 |
+---------------------+--------------------------------------+
```

### Add IP floating cho VM
```
root@controller:~# openstack server add floating ip VM-test-key 192.168.238.155
root@controller:~# openstack server list
+--------------------------------------+-------------+--------+-----------------------------------------+--------------------------+
| ID                                   | Name        | Status | Networks                                | Image Name               |
+--------------------------------------+-------------+--------+-----------------------------------------+--------------------------+
| 881ff1b4-69ff-4d0c-a0ae-a4123f634b08 | VM-test-key | ACTIVE | selfservice=10.10.10.5, 192.168.238.155 | cirros-test-upload-image |
+--------------------------------------+-------------+--------+-----------------------------------------+--------------------------+
```

### Cho phép ping 
```
root@controller:~# openstack security group rule create --proto icmp default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-09-04T14:39:11Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | c7a6dc0c-460e-426a-8ae5-d2a052eb07e5 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 5a7d5cc6c723427e8d0394f70201af54     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | df47cbd5-dce1-4302-859b-67334e3c630f |
| updated_at        | 2017-09-04T14:39:11Z                 |
+-------------------+--------------------------------------+

```

### Cho phép ssh 
```
root@controller:~# openstack security group rule create --proto tcp --dst-port 22 default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-09-04T14:40:20Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | c0bc7546-a932-4509-88ec-c50cc2d7e34b |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 5a7d5cc6c723427e8d0394f70201af54     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | df47cbd5-dce1-4302-859b-67334e3c630f |
| updated_at        | 2017-09-04T14:40:20Z                 |
+-------------------+--------------------------------------+
```


### Log vào VM bằng ssh 
```
root@controller:~# ssh -i mykey cirros@192.168.238.155
```
### Xem thông tin VM 
```
root@controller:~# openstack server show cirros
+-----------------------------+-----------------------------------------------------------------+
| Field                       | Value                                                           |
+-----------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig           | AUTO                                                            |
| OS-EXT-AZ:availability_zone | nova                                                            |
| OS-EXT-STS:power_state      | Running                                                         |
| OS-EXT-STS:task_state       | None                                                            |
| OS-EXT-STS:vm_state         | active                                                          |
| OS-SRV-USG:launched_at      | 2017-09-05T02:35:55.000000                                      |
| OS-SRV-USG:terminated_at    | None                                                            |
| accessIPv4                  |                                                                 |
| accessIPv6                  |                                                                 |
| addresses                   | selfservice=10.10.10.9, 192.168.238.151                         |
| config_drive                |                                                                 |
| created                     | 2017-09-05T02:35:49Z                                            |
| flavor                      | m1.nano (0)                                                     |
| hostId                      | 5eff11ba74fcf6cc1bceb1ab994aee03b2995d4a014c5c63bc72aacf        |
| id                          | 12b5018e-1299-494e-a5d5-2c9a6067325d                            |
| image                       | cirros-test-upload-image (c420b371-7cb3-41d2-abc6-a4bb56c420e5) |
| key_name                    | mykey                                                           |
| name                        | cirros                                                          |
| progress                    | 0                                                               |
| project_id                  | 5a7d5cc6c723427e8d0394f70201af54                                |
| properties                  |                                                                 |
| security_groups             | name='default'                                                  |
| status                      | ACTIVE                                                          |
| updated                     | 2017-09-05T02:35:55Z                                            |
| user_id                     | aa638175684b41e1b0473fbe4b90bd6e                                |
| volumes_attached            |                                                                 |
+-----------------------------+-----------------------------------------------------------------+
```

### Tắt VM
```
root@controller:~# openstack server stop cirros
root@controller:~# openstack server list
+--------------------------------------+-------------+---------+-----------------------------------------+--------------------------+
| ID                                   | Name        | Status  | Networks                                | Image Name               |
+--------------------------------------+-------------+---------+-----------------------------------------+--------------------------+
| 12b5018e-1299-494e-a5d5-2c9a6067325d | cirros      | SHUTOFF | selfservice=10.10.10.9, 192.168.238.151 | cirros-test-upload-image |
| 881ff1b4-69ff-4d0c-a0ae-a4123f634b08 | VM-test-key | ACTIVE  | selfservice=10.10.10.5, 192.168.238.155 | cirros-test-upload-image |
+--------------------------------------+-------------+---------+-----------------------------------------+--------------------------+
```

### Bật VM
```
root@controller:~# openstack server start cirros
```

### Reboot VM
```
root@controller:~# openstack server reboot cirros
```

### Delete VM
```
root@controller:~# openstack server delete cirros
```

## Quota

Quota quy định lượng tài nguyên cho mỗi project. Các thông số được cấu hình trong section [quota] trong file `/etc/nova/nova.conf`

Xem thông tin quota 

```
root@controller:~# openstack quota show
+----------------------+----------------------------------+
| Field                | Value                            |
+----------------------+----------------------------------+
| backup-gigabytes     | 1000                             |
| backups              | 10                               |
| cores                | 20                               |
| fixed-ips            | -1                               |
| floating-ips         | 50                               |
| gigabytes            | 1000                             |
| health_monitors      | None                             |
| injected-file-size   | 10240                            |
| injected-files       | 5                                |
| injected-path-size   | 255                              |
| instances            | 10                               |
| key-pairs            | 100                              |
| l7_policies          | None                             |
| listeners            | None                             |
| load_balancers       | None                             |
| location             | None                             |
| name                 | None                             |
| networks             | 10                               |
| per-volume-gigabytes | -1                               |
| pools                | None                             |
| ports                | 50                               |
| project              | 5a7d5cc6c723427e8d0394f70201af54 |
| project_id           | None                             |
| properties           | 128                              |
| ram                  | 51200                            |
| rbac_policies        | 10                               |
| routers              | 10                               |
| secgroup-rules       | 100                              |
| secgroups            | 10                               |
| server-group-members | 10                               |
| server-groups        | 10                               |
| snapshots            | 10                               |
| subnet_pools         | -1                               |
| subnets              | 10                               |
| volumes              | 10                               |
+----------------------+----------------------------------+
```




 