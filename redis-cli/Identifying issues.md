# Identifying Issues

**Install redis-cli**
```
$ apt-get install redis-tools
```

### Kiểm tra kết nối tới redis
```
$ redis-cli -h <host> -p 6379 -a <password> -n <db> PING
PONG
```

### Xác định lỗi khi redis chiếm quá nhiều RAM
**Kiểm tra keyspace**
```
$ redis-cli INFO keyspace

# Keyspace
db0:keys=120,expires=10,avg_ttl=3600000
db1:keys=500,expires=0,avg_ttl=0
```
Kết quả trả ra:  
- keys: Số lượng key.
- expires: Số key có TTL (hạn dùng).
- avg_ttl: TTL trung bình (ms).

**Kiểm tra memory**
```
$ redis-cli INFO memory

used_memory:10485760
used_memory_human:10M
maxmemory:524288000
```

Kết quả trả ra: 
- used_memory: Bộ nhớ Redis đang sử dụng.
- maxmemory: Giới hạn bộ nhớ (nếu được cấu hình).

**Tìm kiếm keys lớn nhất và sử dụng nhiều nhất**
```
redis-cli -h <host> -p 6379 -a <password> -n <db> --bigkeys
redis-cli -h <host> -p 6379 -a <password> -n <db> --memkeys 100 --memkeys-samples 20
redis-cli -h <host> -p 6379 -a <password> -n <db> --hotkeys
```

Sử dụng bigkeys và memkeys để tìm key lớn nhất trong mỗi DB, hotkeys để tìm kiếm key được truy cập nhiều  
Sự khác nhau giữa bigkeys và memkeys  
| Tính năng         | `--bigkeys`                 | `--memkeys`               | `--hotkeys`                   |
|--------------------|-----------------------------|---------------------------|-------------------------------|
| **Tập trung vào** | Dữ liệu lớn nhất theo loại  | Key chiếm nhiều RAM nhất  | Key được truy cập nhiều nhất |
| **Mục đích chính**| Phân tích kích thước dữ liệu| Phân tích bộ nhớ          | Phân tích truy cập (read/write) |
| **Dùng khi nào**  | RAM đầy                     | RAM đầy, cần tối ưu hóa   | Redis quá tải do truy cập    |
| **Quét key**      | Tất cả key                  | Giới hạn số lượng         | Chỉ theo dõi key truy cập nhiều |
| **Ví dụ kết quả** | Biggest string: key1        | Key: key2, Size: 1024 bytes | Key: key3, Hits: 5000        |


**Lấy thông tin key lớn nhất**  
Đối với các key dạng hash hoặc list, lệnh --bigkeys chỉ đưa ra số item trong key thay vì mem đã sử dụng. Dùng lệnh sau để lấy mem usaged của key
```
redis-cli MEMORY USAGE <key>
```