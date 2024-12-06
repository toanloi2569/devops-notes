# Notes khi sử dụng Kafka

### Kiểm tra connection
**Telnet**  
Sử dụng telnet để kiểm tra kết nối với kafka  
```
telnet <ip> <port>
```


**Kafkacat**
Sử dụng kafkacat để kiểm tra kafka  

1. Cài đặt [kafkacat](https://github.com/edenhill/kcat)   
   ```
   apt-get install kafkacat
   ```

2. Liệt kê danh sách broker và topic  
   ```
   kcat -L -b <ip>:<port>  
   kcat -L -b 10.13.0.45:9092
   ```

3. Kiểm tra kết nối producer và broker   
   Khi đã có danh sách topic, có thể test producer bằng cách gửi 1 message tới topic bằng lệnh sau. Kcat sẽ cho phép người dùng nhập message. Nhập message, `ctrl+D` để thoát  
   ```
   kcat -b <ip>:<port> -P -t <topic name>
   ```

4. Kiểm tra kết nối consumer và broker  
   Khi đã gửi message tới topic. Kiểm tra topic đã có message chưa bằng lệnh sau. Nếu consumer kết nối được tới broker, kcat sẽ trả ra danh sách message  
   ```
   kcat -b <ip>:<port> -C -t <topic name>
   ```
