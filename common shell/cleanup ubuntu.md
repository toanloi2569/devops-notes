# Cleanup Ubuntu  
Tham khảo từ blogs [6 Commands to Clean Up Your Ubuntu System From the Terminal](https://www.howtogeek.com/commands-to-clean-up-your-ubuntu-system-from-the-terminal/?ref=dailydev)  

### Uninstall Programs
Liệt kê các package đã install bằng một trong 2 lệnh sau:
```
$ dpkg --list
$ apt list --installed
```
  
### Cleanup apt cache
Kiểm tra dung lượng cache của apt bằng `du` và vị trí apt lưu cache, thường là `/var/cache/apt`
```
$ sudo du -sh /var/cache/apt
[sudo] password for toanloi: 
223M    /var/cache/apt
```

Cleanup cache bằng lệnh sau
```
$ sudo apt-get clean
```

### Remove Packages You No Longer Need
Sử dụng lệnh sau để xóa package không sử dụng
```
$ sudo apt-get autoremove
```

### Clean Up Journal Logs
Log hệ thống sẽ không bao giờ bị xóa và tạo nên 1 lượng lớn dữ liệu, kiểm tra disk space dùng để lưu logs hệ thống bằng lệnh sau:
```
$ journalctl --disk-usage
Archived and active journals take up 4.0G in the file system.
```

Sử dụng lệnh sau để xóa log, tham số vacuum-time biểu thị thời gian muốn giữ lại log. Trong ví dụ, mình đang muốn giữ lại log trong 7 ngày gần nhất.
```
$ sudo journalctl --vacuum-time=7d
```

### Clear Your Thumbnail Cache
Mỗi khi add 1 ảnh, ubuntu tự tạo 1 thumbnail để dễ dàng xem trong trình quản lý file. Sử dụng `du` để kiểm tra dung lượng đang sử dụng và `rm -rf` để xóa
```
$ du -sh ~/.cache/thumbnails
225M    /home/toanloi/.cache/thumbnails

$ rm -rf ~/.cache/thumbnails/*
```
