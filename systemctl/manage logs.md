# systemctl logs: A Guide to Managing Logs in Linux

Dịch từ blog [systemctl logs: A Guide to Managing Logs in Linux](https://last9.io/blog/systemctl-logs/?ref=dailydev)

### Nhật ký systemctl là gì?
systemctl là một công cụ được sử dụng để kiểm soát hệ thống và trình quản lý dịch vụ systemd. Đây là một tiện ích quan trọng trong các bản phân phối Linux hiện đại sử dụng systemd để khởi tạo hệ thống, quản lý tiến trình và xử lý dịch vụ.

Khi nói đến nhật ký, systemctl tương tác với journald, một dịch vụ ghi nhật ký của systemd. Điều này cho phép bạn xem, lọc và quản lý nhật ký cho các dịch vụ đang chạy trên hệ thống của mình.

### Common journalctl Commands
Dưới đây là các journalctl Commands thường được sử dụng để quản lý và xem logs

| Commmand                              | Description                                                                       |
|-----------------------------------|-----------------------------------------------------------------------------|
| `journalctl`                      | Xem tất cả nhật ký hệ thống theo thứ tự thời gian ngược.                |
| `journalctl -u <service-name>`    | Xem nhật ký của một dịch vụ cụ thể (ví dụ: `journalctl -u nginx`).           |
| `journalctl -f`                   | Xem nhật ký theo thời gian thực (tương tự `tail -f`).                       |
| `journalctl --since "YYYY-MM-DD"` | Xem nhật ký từ một ngày cụ thể (ví dụ: `journalctl --since "2024-12-01"`).  |
| `journalctl -b`                   | Xem nhật ký từ phiên khởi động hiện tại.                                    |
| `journalctl -p <priority>`        | Lọc nhật ký theo mức ưu tiên (ví dụ: `journalctl -p err` để xem lỗi).       |
| `journalctl -n <number>`          | Hiển thị số lượng nhật ký gần nhất được chỉ định (ví dụ: `journalctl -n 100`).|
| `journalctl --vacuum-time=2weeks` | Dọn dẹp nhật ký cũ hơn thời gian được chỉ định (ví dụ: hai tuần).           |
| `journalctl --vacuum-size=500M`   | Dọn dẹp nhật ký để giữ kích thước nhật ký hệ thống dưới giới hạn được đặt.   |


### How to View systemctl Logs

**View All Logs**
```
journalctl
```

**View Logs for a Specific Service**
```
journalctl -u nginx
```

**View Logs in Real-Time**
```
journalctl -u nginx -f
```

**View Logs for a Specific Time Period**
```
journalctl --since "2024-12-01" --until "2024-12-09"
```

**Show Logs for Boot Sessions**  
View log từ boot session hiện tại
```
journalctl -b
```

View log từ boot session trước
```
journalctl -b -1
```


### Filtering Logs for Efficiency

**Filtering by Priority Level**  
Logs được phân loại theo mức độ ưu tiên (0: emergency, 7: debug). Để filter message quan trọng:
```
journalctl -p err
```

**Filtering by Keywords**
```
journalctl -u nginx | grep "connection failed"
```

**Limiting Output**
```
journalctl -n 50
```

### Best Practices for Managing systemctl Logs

**Log Rotation**  
Rotate log bằng cách modify file `/etc/systemd/journald.conf`  
```
[Journal]
SystemMaxUse=1G
```

**Archiving Logs**  
Để lưu trữ lâu dài, hãy định cấu hình systemd để chuyển tiếp nhật ký đến máy chủ nhật ký hệ thống bên ngoài hoặc sử dụng các công cụ như Fluentd hoặc Logstash.  

**Regular Log Cleanup**  
Có thể clear log theo thời gian hoặc disk usage
```
journalctl --vacuum-time=2weeks
journalctl --vacuum-size=500M
```

**Monitoring and Alerting**  
Thiết lập các công cụ giám sát như Prometheus để ghi logs và trigger alert dựa trên các patterns cụ thể.


### Troubleshooting with systemctl Logs
Khi xử lý sự cố với các dịch vụ, nhật ký systemctl thường là nơi đầu tiên cần kiểm tra. Dưới đây là một số tình huống mà các nhật ký này có thể hữu ích:

- Sự cố dịch vụ: Nếu một dịch vụ không khởi động được, nhật ký systemctl thường cung cấp các thông báo lỗi chỉ ra chính xác vấn đề, chẳng hạn như cấu hình sai hoặc thiếu phụ thuộc.

- Vấn đề hiệu suất: Việc sử dụng tài nguyên cao hoặc hiệu suất chậm có thể được truy vết thông qua các vấn đề được ghi nhận trong nhật ký của dịch vụ. Kiểm tra nhật ký có thể tiết lộ liệu một dịch vụ có đang sử dụng quá nhiều tài nguyên hoặc gặp lỗi hay không.

- Hệ thống bị treo: Nhật ký cung cấp các thông tin quan trọng về các sự cố treo hệ thống hoặc kernel panic, giúp quản trị viên xác định liệu vấn đề là do phần cứng hay lỗi phần mềm gây ra.

### Security and Permissions

**Understanding Log Access Control**  
Nhật ký hệ thống thường chứa thông tin nhạy cảm. Hiểu rõ ai có thể truy cập những nhật ký này là điều quan trọng để đảm bảo an ninh hệ thống. Dưới đây là những điều bạn cần biết:

```
# View current log access permissions
ls -l /var/log/journal/

# Add user to systemd-journal group
sudo usermod -a -G systemd-journal username

# Verify access
groups username
```

**Understanding Log Impact on System Resources**  


Disk Usage Monitoring
```
# Check journal disk usage
journalctl --disk-usage

# Monitor real-time disk writes
iotop -o
```

Memory Impact
```
# Check journal memory usage
systemctl status systemd-journald
```