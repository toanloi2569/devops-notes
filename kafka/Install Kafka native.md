# Cài đặt kafka môi trường production
Cài đặt theo link [Apace Kafka Quickstart](https://kafka.apache.org/quickstart)

### Pre Requisite  
Update OS và cài đặt Java 11  
```aiignore
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```

### Get kafka  
[Download](https://kafka.apache.org/downloads) kafka và giải nén
```aiignore
wget https://dlcdn.apache.org/kafka/3.9.0/kafka_2.12-3.9.0.tgz
tar -xzf kafka_2.12-3.9.0.tgz
mv kafka_2.12-3.9.0 /opt/kafka
```

### Config Kafka
```aiignore
mkdir /opt/kafka/data
mkdir /opt/kafka/logs_metadata

cd /opt/kafka/config/kraft/
cp server.properties server.properties.bak
vi server.properties
```

Sửa các thông số sau trong file `server.properties`  
```aiignore
    node.id=1
    num.network.threads=3
    num.io.threads=8
    log.dirs=/opt/kafka/data
    metadata.log.dir=/opt/kafka/logs_metadata
 
 
    process.roles=broker,controller
    listeners=BROKER://<internal_ip>:9092,CONTROLLER://<internal_ip>:9093, INTERNAL://<internal_ip>:19092
    advertised.listeners=BROKER://<external_ip>:9092, INTERNAL://<internal_ip>:19092
    listener.security.protocol.map=BROKER:SASL_PLAINTEXT,CONTROLLER:SASL_PLAINTEXT, INTERNAL:SASL_PLAINTEXT
    controller.quorum.voters=1@server1ip:9093,2@server2ip:9093,3@server3ip:9093
     
    inter.broker.listener.name=INTERNAL
    controller.listener.names=CONTROLLER
    
    listener.name.broker.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
        username="admin" \
        password="password" \
        user_admin="password";
    
    listener.name.controller.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
        username="admin" \
        password="password" \
        user_admin="password";
    
    listener.name.internal.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
        username="admin" \
        password="password" \
        user_admin="password";
    
    sasl.enabled.mechanisms=PLAIN
    sasl.mechanism.controller.protocol=PLAIN
    sasl.mechanism.inter.broker.protocol=PLAIN
     
    authorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer
    allow.everyone.if.no.acl.found=false
    super.users=User:admin
     
    delete.topic.enable=true
    socket.send.buffer.bytes=1048576
    socket.receive.buffer.bytes=1048576
    socket.request.max.bytes=104857600
     
    num.partitions=3 # số partition mặc định khi tạo topic, nếu có 1 node thì chỉ được 1 partition
    default.replication.factor=2 # số replica mặc định khi tạo topic, nếu có 1 node thì chỉ được 1 replica
    min.insync.replicas=2
    log.retention.hours=168
    log.segment.bytes=1073741824
    log.retention.check.interval.ms=300000
    auto.create.topics.enable=true
    unclean.leader.election.enable=false
```

Tạo file `jaas.conf` trong thư mục `/opt/kafka/config/kraft/`
```aiignore
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="password"
    user_admin="password";
};
```

Tạo Kafka cluster ID
```aiignore
bash /opt/kafka/bin/kafka-storage.sh random-uuid
```

Sử dụng cùng giá trị UUID này cho các node khác trong cluster
```aiignore
bash /opt/kafka/bin/kafka-storage.sh format -t GeneratedUUID -c /opt/kafka/config/kraft/server.properties
```

### Tạo file systemd service
Tạo file service
```aiignore
vi /lib/systemd/system/kafka.service
```

Nội dung file
```aiignore
[Unit]
Description=Apache Kafka server
After=network-online.target
Requires=network-online.target

[Service]
Type=simple
Restart=on-failure


User=root
Group=root
SyslogIdentifier=kafka
Environment="KAFKA_HEAP_OPTS=-Xms1G -Xmx1G"
Environment="KAFKA_OPTS=-Djava.security.auth.login.config=/opt/kafka/config/kraft/jaas.config"

ExecStart=/bin/sh -c '/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties > /opt/kafka/logs/kafka.log 2>&1'
ExecStop=/bin/sh -c '/opt/kafka/bin/kafka-server-stop.sh /opt/kafka/config/kraft/server.properties > /opt/kafka/logs/kafka.log 2>&1'
WorkingDirectory=/opt/kafka

[Install]
WantedBy=multi-user.target
```
   
### Start và enable service
```aiignore
systemctl daemon-reload
systemctl enable --now kafka
```

### Kiểm tra service
Cài đặt `kafkacat` để kiểm tra kafka  
```aiignore
apt-get install kafkacat
```

Đối với các node cùng cụm Kafka, sử dụng INTERNAL IP để kết nối. Đối với các node ngoài cụm, sử dụng EXTERNAL IP     
Liệt kê danh sách broker và topic  
```aiignore
# Không cần authentication
kcat -L -b <ip>:<port>

# Sử dụng authentication SASL_PLAINTEXT
kcat -L -b <ip>:<port> -X security.protocol=SASL_PLAINTEXT -X sasl.mechanisms=PLAIN -X sasl.username=admin -X sasl.password=password 
```

Kiểm tra kết nối producer và broker  
```aiignore
# Không cần authentication
kcat -b <ip>:<port> -P -t <topic name>

# Sử dụng authentication SASL_PLAINTEXT
kcat -b <ip>:<port> -X security.protocol=SASL_PLAINTEXT -X sasl.mechanisms=PLAIN -X sasl.username=admin -X sasl.password=password -P -t <topic name>

# Sau nhập message, `ctrl+D` để thoát
```

Kiểm tra kết nối consumer và broker  
```aiignore
# Không cần authentication
kcat -b <ip>:<port> -C -t <topic name>

# Sử dụng authentication SASL_PLAINTEXT
kcat -b <ip>:<port> -X security.protocol=SASL_PLAINTEXT -X sasl.mechanisms=PLAIN -X sasl.username=admin -X sasl.password=password -C -t <topic name>
```


