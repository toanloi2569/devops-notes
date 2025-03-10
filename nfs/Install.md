# Install NFS
Tham khảo từ bài viết [NFS là gì? hướng dẫn cài đặt và sử dụng NFS trên Linux](https://azdigi.com/blog/linux-server/linux-can-ban/nfs-la-gi-huong-dan-cai-dat-va-su-dung-nfs-tren-linux/)

Giả sử ta có 4 VM như sau 
```
10.50.11.2: Server
10.50.11.3: Client
10.50.11.4: Client
10.50.11.5: Client
``` 
Ta tiến hành cài đặt NFS để lưu file tập trung từ client vào server

### Cài đặt NFS Server trên Ubuntu/Debian
Chạy lênh sau trên máy server để cài đặt và kiểm tra trạng thái server
```
apt-get update && apt-get upgrade
apt install nfs-kernel-server
systemctl status nfs-server
```

**Tạo thư mục chia sẻ**
Tạo thư mục để lưu các file sẽ chia sẻ cho các NFS Client  
`mkdir /mnt/nfs-server`

**Cấu hình thư mục chia sẻ**
Tạo file **`/etc/exports`** để lưu config file chia sẻ cho NFS Client. Ví dụ chia sẻ thư mục `/mnt/nfs` cho ip `10.50.11.3`, `10.50.11.4`, `10.50.11.5`, ta tạo file config với nội dung như sau:

```
/mnt/nfs-server 10.50.11.3(rw,sync,no_subtree_check,no_root_squash)
/mnt/nfs-server 10.50.11.4(rw,sync,no_subtree_check,no_root_squash)
/mnt/nfs-server 10.50.11.5(rw,sync,no_subtree_check,no_root_squash)
```

**Các quyền trong NFS**
Các quyền config trong NFS bao gồm
1. ro: Read only
2. rw: Read – write
3. noaccess: Denied access
4. root_squash: Ngăn remote root users
5. no_root_squash: Cho phép remote root users
6. no_subtree_check: Không kiểm tra subtree

Sau khi cấu hình quyền, chạy `exportfs -a` để re-apply cấu hình 

Khởi động lại NFS server để nhận config mới
`systemctl restart nfs-kernel-server`


### Cấu hình NFS ở client

**Cài đặt NFS client**
Chạy lệnh sau để cài đặt NFS client trên máy Client
`apt-get install nfs-common`

**Cấu hình thư mục chia sẻ**
Tạo 1 folder để client có thể mount vào thư mục gốc trên server
`mkdir /mnt/nfs-client`

Thực hiện lệnh mount tới Server
`mount 10.50.11.2:/mnt/nfs-server /mnt/nfs-client`

Tuy nhiên, nếu như chúng ta chỉ thực thi lệnh mount trên thì sẽ bị mất khi máy tính, VPS hoặc Server được khởi động lại. Do đó cần thực hiện mount tự động khi máy tính, VPS hoặc Server được khởi động lại, ta cần thêm dòng sau vào file /etc/fstab:
`10.50.11.2:/mnt/nfs-server /mnt/nfs-client nfs rw,sync,hard,intr 0 0`

### Kiểm tra hoạt động nfs
Thực hiện tạo file `test-nfs.txt` trên server và kiểm tra xem đã có thông tin ở client chưa.
