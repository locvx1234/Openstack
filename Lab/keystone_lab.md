Trước khi làm việc với Keystone, cần khai báo các biến môi trường để xác thực.

$ . admin-openrc  

## Lấy token 

```
root@controller:~# openstack token issue
```

```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-08-15T08:07:02+0000                                                                                                                                                                |
| id         | gAAAAABZkp2WB9Isj8taKOrQfVg_wJKqhS-eRDlTtc6WLKja5tVTHZtv0vA5RzG3MnxiBQfsZRR0ngEX5TN2o36IzBFe-dOzD_DCykziOidYEnZHe5kW6yxlru1B7KrmVZFNL5tP137FDIcdy9qOiasYS4i8ZChX64cFw89xcAa8zKr6ljXq-Dw |
| project_id | 1e1f11c83cf24288b10524f3a05878e6                                                                                                                                                        |
| user_id    | 9c57a9c095ad42d1be6448e1452af154                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
Lấy token và các thông tin khác 
```
root@controller:~# curl -i -X POST -H "Content-Type: application/json" -d '
  {
    "auth": {
      "identity": {
        "methods": [
          "password"
        ],
        "password": {
          "user": {
            "name": "admin",
            "password": "locvx1234",
            "domain": {
              "name": "Default"
            }
          }
        }
      },
      "scope": {
        "project": {
          "name": "admin",
          "domain": {
            "name": "Default"
          }
        }
      }
    }
  }' http://10.10.10.10:5000/v3/auth/tokens
```

Token là giá trị trường `id`

Để sử dụng token 

```
export OS_TOKEN=gAAAAABZkp2WB9Isj8taKOrQfVg_wJKqhS-eRDlTtc6WLKja5tVTHZtv0vA5RzG3MnxiBQfsZRR0ngEX5TN2o36IzBFe-dOzD_DCykziOidYEnZHe5kW6yxlru1B7KrmVZFNL5tP137FDIcdy9qOiasYS4i8ZChX64cFw89xcAa8zKr6ljXq-Dw
```


## Liệt kê 

- User 
```
root@controller:~# openstack user list
```

```
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 3b3ecc5e7c7a4943892a603bd9054fa7 | placement |
| 9c57a9c095ad42d1be6448e1452af154 | admin     |
| a4daeb58849742b79b683e243324e215 | nova      |
| bee1ae76dfbc4e749fdbd2171b73ea88 | demo      |
| cb4a0fb018fb432180ad445b97811221 | glance    |
+----------------------------------+-----------+

```

hoặc
```
root@controller:~#  curl -s -H "X-Auth-Token: $OS_TOKEN"   http://10.10.10.10:5000/v3/users | python -mjson.tool
```
- Project 
```
root@controller:~# openstack project list
```
hoặc 

```
root@controller:~# curl -s -H "X-Auth-Token: $OS_TOKEN"   http://10.10.10.10:5000/v3/projects | python -mjson.tool
```

- Group
```
root@controller:~# openstack group list
```
hoặc 
```
root@controller:~# curl -s -H "X-Auth-Token: $OS_TOKEN"   http://10.10.10.10:5000/v3/groups | python -mjson.tool
```

- Role
```
root@controller:~# openstack role list
```
hoặc
```
root@controller:~# curl -s -H "X-Auth-Token: $OS_TOKEN"   http://10.10.10.10:5000/v3/roles | python -mjson.tool
```

- Domain

```
root@controller:~# openstack domain list
```
hoặc 
```
root@controller:~# curl -s -H "X-Auth-Token: $OS_TOKEN"   http://10.10.10.10:5000/v3/domains | python -mjson.tool
```


## Tạo mới 

- Domain 
```
root@controller:~# openstack domain create new_domain
```
hoặc
```
root@controller:~# curl -s -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{ "domain": { "name": "new_domain2"}}' http://10.10.10.10:5000/v3/domains | python -mjson.tool
```

- Project 
```
root@controller:~# openstack project create --domain new_domain --description "New project in domain new_domain" new_project
```
hoặc 
```
root@controller:~# curl -s  -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json" -d '{ "project": { "name": "new_project2", "domain_id": "ID_DOMAIN", "description": "New project2 in domain new_domain2"}}' http://10.10.10.10:5000/v3/projects | python -mjson.tool
```

- User 
```
root@controller:~# openstack user create --domain new_domain --password new_passwd new_user
```
hoặc 
```
root@controller:~# curl -s  -H "X-Auth-Token: $OS_TOKEN" -H "Content-Type: application/json"  -d '{ "user": { "name": "new_user", "password": "new_passwd", "domain_id": "ID_DOMAIN", "description": "new_user in domain new_domain2"}}' http://10.10.10.10:5000/v3/users | python -mjson.tool
```


- Add role 
```
openstack role add --project new_project --project-domain new_domain --user new_user --user-domain new_domain user
```

- Liệt kê role 

```
openstack role list --user new_user --project new_project
```



