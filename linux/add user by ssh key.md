# Các bước thêm user vào máy chủ linux bằng SSH key

### 1. Tạo user mới
```bash
useradd -m -s /bin/bash <username>

# -m: Tạo thư mục home cho user
# -s /bin/bash: Chỉ định shell mặc định cho user
```

### 2. Tạo thư mục `.ssh` cho user
```bash
mkdir /home/<username>/.ssh
chown <username>:<username> /home/<username>/.ssh
chmod 700 /home/<username>/.ssh
```

### 3. Tạo file `authorized_keys` trong thư mục `.ssh`
Nếu đã có public key, copy nội dung vào file `authorized_keys`
```bash
vi /home/<username>/.ssh/authorized_keys
```

Nếu đã có file public (ví dụ: `id_rsa.pub`), copy nội dung vào file `authorized_keys`
```bash
cat id_rsa.pub >> /home/<username>/.ssh/authorized_keys
```

### 4. Phân quyền cho file `authorized_keys`
```bash
chown <username>:<username> /home/<username>/.ssh/authorized_keys
chmod 600 /home/<username>/.ssh/authorized_keys
```

### 5. Đặt mật khẩu cho user
```bash
passwd <username>
```

Trường hợp không muốn đặt mật khẩu, có thể sử dụng lệnh sau để vô hiệu hóa mật khẩu
```bash
passwd -d <username>
```

hoặc chỉnh sửa visudo để cho phép user đó sử dụng sudo mà không cần mật khẩu
```bash
visudo

# Thêm dòng sau vào file
<username> ALL=(ALL) NOPASSWD: ALL
```