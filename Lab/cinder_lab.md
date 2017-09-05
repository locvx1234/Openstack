## Tạo volume 

### Tạo volume no-source 

Volume tạo theo cách này là volume trống, gọi là non-bootable

```
root@controller:~# openstack volume create --size 1 volume1
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2017-09-05T08:34:39.934834           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | 2bd782e2-07aa-4eb6-b677-478073b44fb9 |
| multiattach         | False                                |
| name                | volume1                              |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 1                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | None                                 |
| updated_at          | None                                 |
| user_id             | aa638175684b41e1b0473fbe4b90bd6e     |
+---------------------+--------------------------------------+
```

hoặc 

```
root@controller:~# cinder create --display-name volume2 1
+------------------------------+--------------------------------------+
| Property                     | Value                                |
+------------------------------+--------------------------------------+
| attachments                  | []                                   |
| availability_zone            | nova                                 |
| bootable                     | false                                |
| consistencygroup_id          | None                                 |
| created_at                   | 2017-09-05T08:35:18.000000           |
| description                  | None                                 |
| encrypted                    | False                                |
| id                           | d0411a2e-2f7a-434c-99db-60ec430d1a24 |
| metadata                     | {}                                   |
| multiattach                  | False                                |
| name                         | volume2                              |
| os-vol-tenant-attr:tenant_id | 5a7d5cc6c723427e8d0394f70201af54     |
| replication_status           | None                                 |
| size                         | 1                                    |
| snapshot_id                  | None                                 |
| source_volid                 | None                                 |
| status                       | error                                |
| updated_at                   | 2017-09-05T08:35:18.000000           |
| user_id                      | aa638175684b41e1b0473fbe4b90bd6e     |
| volume_type                  | None                                 |
+------------------------------+--------------------------------------+
```

### Liệt kê volume 
```
root@controller:~# cinder list

```
hoặc
```
root@controller:~# openstack volume list

```

