Trước khi làm việc với Keystone, cần khai báo các biến môi trường để xác thực.

$ . admin-openrc  

## Lấy token 

```
root@controller:~# openstack token issue
```

Lấy token và các thông tin khác 
```
root@controller:~# curl -i -X POST -H "Content-Type: application/json" -d '
>  {
>    "auth": {
>      "identity": {
>        "methods": [
>          "password"
>        ],
>        "password": {
>          "user": {
>            "name": "admin",
>            "password": "locvx1234",
>            "domain": {
>              "name": "Default"
>            }
>          }
>        }
>      },
>      "scope": {
>        "project": {
>          "name": "admin",
>          "domain": {
>            "name": "Default"
>          }
>        }
>      }
>    }
>  }' http://10.10.10.10:5000/v3/auth/tokens
```

## Liệt kê 

- User 
```
root@controller:~# openstack user list
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

hoặc 

...




