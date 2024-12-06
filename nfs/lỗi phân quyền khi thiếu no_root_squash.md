# Lỗi phân quyền khi thiếu no_root_squash

### Usecase
Mình đã gặp lỗi phân quyền ghi file khi sử dụng NFS và k8s. Cụ thể usecase như sau:

Mình sử dụng NFS để ghi log tập trung trên cụm k8s

1. Cụm k8s có 3 node trên 3 VM khác nhau: VM01, VM02, VM03
2. Service gồm 3 pod cài đặt trên 3 node, log được ghi ra file và mount vào VM
3. NFS server cài trên VM01, NFS client được cài trên VM02 và VM03

Service sẽ ghi log vào file trong pod -> File trong pod được mount vào file trên VMs -> File trên VMs ghi vào file log tập trung trên NFS server.

Config nfs như sau
```
/mnt/nfs-server 10.50.11.3(rw,sync,no_subtree_check)
/mnt/nfs-server 10.50.11.4(rw,sync,no_subtree_check)
/mnt/nfs-server 10.50.11.5(rw,sync,no_subtree_check)
```

Vấn đề: Sau khi setup, service trên VM01, VM02 không thể ghi log vào file NFS. 


### Cơ chế root squash
Root squash là một tính năng bảo mật trong NFS nhằm giảm thiểu nguy cơ truy cập trái phép với quyền root trên máy chủ NFS từ các máy client.

**Lợi ích của Root Squash**
1. Ngăn chặn đặc quyền superuser:
   Khi root squash được kích hoạt (mặc định trong hầu hết các cấu hình NFS), bất kỳ yêu cầu truy cập nào từ người dùng root (UID 0) trên máy client NFS sẽ được ánh xạ thành một người dùng không có đặc quyền (thường là nobody hoặc nfsnobody) trên máy chủ NFS. Điều này ngăn cản người dùng root trên máy client có được quyền superuser trên máy chủ NFS, hạn chế khả năng sửa đổi hoặc tương tác với các file và thư mục chia sẻ.

2. Giảm nguy cơ truy cập trái phép:
   Nếu một máy client bị tấn công, đặc biệt là khi tài khoản root trên máy đó bị xâm phạm, root squash sẽ giúp ngăn chặn kẻ tấn công sử dụng NFS để đạt được quyền truy cập root vào các thư mục chia sẻ trên máy chủ NFS.

3. Duy trì bảo mật file:
   Bằng cách squash quyền root, NFS đảm bảo tính bảo mật và toàn vẹn của các quyền file và quyền sở hữu trên máy chủ. Điều này đảm bảo rằng các chia sẻ NFS tuân thủ nguyên tắc "ít quyền nhất" (least privilege), chỉ cho phép truy cập và sửa đổi trong phạm vi mà quản trị viên của máy chủ đã định sẵn.


### Giải thích
Khi service ở VM03 tạo file, VM03 tạo file root:root.

Khi sửa trực tiếp file trên VM01 và VM02, NFS sử dụng cơ chế "user squashing", chuyển file thành permission nobody:nogroup, NFS client từ VM01 và VM02 cũng nhìn file với permission là nobody:nogroup. 

Khi thực hiện từ k8s, NFS không thực hiện cơ chế "user squashing", dẫn đến file vẫn có permission là root:root, nhưng VM01 và VM02 chỉ truy cập dưới quyền nobody:nogroup -> Không có permission.

### Solution
Hiện đã hotfix bằng việc thêm no_root_squash, tuy nhiên việc này không được khuyến khích và hacker có thể khai thác chúng như sau ([link](https://cybergrover.medium.com/mastering-nfs-shares-setup-root-squash-and-nmap-enumeration-demystified-4e4630bf8fb1ư)):
1. Giành quyền root trên máy chủ NFS:
   Nếu một kẻ tấn công giành được quyền root trên máy client NFS và tùy chọn no_root_squash được kích hoạt, chúng có thể truy cập và sửa đổi các file trên máy chủ NFS với quyền root. Điều này có thể dẫn đến việc thay đổi các file quan trọng, xóa dữ liệu hoặc cài đặt phần mềm độc hại.

2. Nâng cấp đặc quyền:
   Kẻ tấn công có quyền truy cập không phải root trên máy client có thể khai thác các lỗ hổng cục bộ để đạt được quyền root, sau đó tận dụng NFS mount với no_root_squash để leo thang đặc quyền trên máy chủ.

3. Vượt qua bảo mật máy chủ:
   Kẻ tấn công có thể bỏ qua các biện pháp bảo mật và quyền hạn đã thiết lập trên máy chủ, vì máy chủ sẽ tin tưởng root từ máy client và coi đây là một thao tác superuser hợp lệ.

