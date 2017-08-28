:arrow_left: [Networking](https://github.com/locvx1234/Openstack/blob/master/Install_guide/Networking.md)

## Dashboard (Horizon)

Cài đặt và cấu hình dashboard trên node Controller 

Dịch vụ core duy nhất  được yêu cầu bởi dashboard là Identity service. Bạn có thể sử dụng dashboard để kết hợp các service khác như Image service, Compute, Networking. 

Bạn cũng có thể sử dụng Dashboard trong môi trường với các service độc lập như Object Storage.

### Cài đặt và cấu hình các thành phần 

```
# apt install openstack-dashboard -y
```

Edit file `/etc/openstack-dashboard/local_settings.py`

```


ALLOWED_HOSTS = ['one.example.com', 'two.example.com']  	# Cho phép các host có thể truy cập Dashboard, mặc định là '*' - all 


OPENSTACK_API_VERSIONS = {						# line 55
	"identity": 3,
	"image": 2,
	"volume": 2,
}

OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True 	# line 66 

OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"   # line 74

SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {					# line 131
	'default': {
		'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
		'LOCATION': 'controller:11211',
	}
}


OPENSTACK_HOST = "controller" # line 161

OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST  # line 162

OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"	    # line 163








```

Nếu sử dụng networking option 1, disable support cho layer 3 netwoking:

```
OPENSTACK_NEUTRON_NETWORK = {
	...
	'enable_router': False,
	'enable_quotas': False,
	'enable_ipv6': False,
	'enable_distributed_router': False,
	'enable_ha_router': False,
	'enable_lb': False,
	'enable_firewall': False,
	'enable_vpn': False,
	'enable_fip_topology_check': False,
}
```

Option : cấu hình Time zone : 

```
	TIME_ZONE = "Asia/Ho_Chi_Minh"
```

```
# chown www-data /var/lib/openstack-dashboard/secret_key
```

Reload cấu hình webserver :

```
# service apache2 reload
```

Sau đó truy cập `http:\\controller/horizon` và đăng nhập với user `admin` hoặc `demo` và domain `default` 


:arrow_right: [Block Storage service](https://github.com/locvx1234/Openstack/blob/master/Install_guide/BlockStorage.md)