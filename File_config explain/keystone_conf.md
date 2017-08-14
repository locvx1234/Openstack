
/etc/keystone/keystone.conf

```
[database]
# Đường dẫn đến server database sẽ được sử dụng cho keystone
connection =  mysql+pymysql://keystone:locvx1234@controller/keystone

[fernet_tokens]
# Thư mục chứa các fernet key. 
# Thư mục này phải tồn tại trước khi chạy lệnh`keystone-manage fernet_setup` 
# Các keys ở trong thư mục này sẽ thuộc 1 trong 3 loại keys:
# - staged key (luôn có chỉ số là 0) được sử dụng để giải mã token
# - primary key (luôn có chỉ số cao nhất) được sử dụng để mã hóa và giải mã token
# - các key còn lại là secondary key được sử dụng cho giải mã token
key_repository = /etc/keystone/fernet-keys/ 

# Số fernet keys tối đa trong thư mục chứa của nó

# Mặc định có tối đa là 3 key. Nghĩa là có thể có 1 staged key (index 0), 1 primary key (index cao nhất), 1 secondary key
# Nếu tăng giá trị này lên, ta có thể có thêm secondary key.
# Minimum value: 1
max_active_keys = 3

[token]
# Khai báo provider dùng để kiểm soát các công việc thực hiện tạo, xác nhận, thu hồi token trong keystone
# Keystone có hai provider là "uuid" và "fernet"
# UUID token sẽ được lưu lại trong backend, để sử dụng uuid, cần chỉ rõ backend (ở trong tùy chọn `[token] driver`)
# fernet token thì sẽ không được lưu lại như uuid, nhưng nó phải được sử dụng khi chạy lệnh `keystone-manage fernet_setup` (và `keystone-manage fernet_rotate`) để quản lý key

provider = fernet

# Lượng thời gian tồn tại của token, kể từ lúc token được tạo
# Đơn vị tính bằng giây (s)
expiration = 300

# Để thực hiện được caching token thì phải `enabled = true` trong section [cache]
caching = true

# Thời gian cache của token
cache_time = <None>

[cache]

# Cho phép bật chức năng cache
enabled = true

```