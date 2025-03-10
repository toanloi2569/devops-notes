<h1 style="color:orange">Note cài đặt keycloak 26</h1>
<h2 style="color:orange">1. Chuẩn bị</h2>
Chuẩn bị DB MySQL cho keycloak<br>
Keycloak được cài đặt trên cụm k8s<br>
Tạo secret cho k8s để pull được image nếu cần thiết

```
kubectl create secret generic pullimgsecret --from-file=.dockerconfigjson=~/.docker/config.json --type=kubernetes.io/dockerconfigjson
```
<h2 style="color:orange">1.1. Cài đặt mysql</h2>
Tạo user mysql cho keycloak:

```
mysql> CREATE USER 'keycloak'@'%' IDENTIFIED BY '<password>';
mysql> GRANT ALL PRIVILEGES ON `keycloak`.* TO `keycloak`@`%`;
mysql> CREATE DATABASE keycloak;
```
Lưu ý: Nếu mysql cài kiểu replication-group chứ không phải master-slave thì khi up keycloak lên sẽ không thể ghi vào db của mysql. Lý do là cài replication group thì mysql sẽ không cho tạo mà không có foreign key. Để fix, ta phải tạo thủ công 1 bảng trong db keycloak:
```
mysql> use keycloak;
mysql> CREATE TABLE `DATABASECHANGELOG` (
    `ID` varchar(255) NOT NULL,
    `AUTHOR` varchar(255) NOT NULL,
    `FILENAME` varchar(255) NOT NULL,
    `DATEEXECUTED` datetime NOT NULL,
    `ORDEREXECUTED` int(11) NOT NULL,
    `EXECTYPE` varchar(10) NOT NULL,
    `MD5SUM` varchar(35) DEFAULT NULL,
    `DESCRIPTION` varchar(255) DEFAULT NULL,
    `COMMENTS` varchar(255) DEFAULT NULL,
    `TAG` varchar(255) DEFAULT NULL,
    `LIQUIBASE` varchar(20) DEFAULT NULL,
    `CONTEXTS` varchar(255) DEFAULT NULL,
    `LABELS` varchar(255) DEFAULT NULL,
    `DEPLOYMENT_ID` varchar(10) DEFAULT NULL,
    PRIMARY KEY (`ID`,`AUTHOR`,`FILENAME`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
Tham khảo: https://mysql-dba.co.uk/2020/11/29/keycloak-and-innodb-cluster/
<h2 style="color:orange">2. Cài đặt</h2>
Apply các file sau vào cụm k8s:

```yaml
    # k apply -f cm.yaml

apiVersion: v1
data:
    KC_PROXY_HEADERS: "xforwarded"
    KC_HOSTNAME: "keycloak.ai"
    KC_HTTP_ENABLED: "true"
    KC_BOOTSTRAP_ADMIN_USERNAME: "admin"
    KC_DB: "mysql"
    KC_DB_SCHEMA: "keycloak"
    KC_DB_USERNAME: "keycloak"
    KC_DB_URL: "jdbc:mysql://x.x.x.x/keycloak"
    KC_HEALTH_ENABLED: "true"
    KC_HTTP_RELATIVE_PATH: "/auth"
    KC_PROXY_ADDRESS_FORWARDING: "true"
kind: ConfigMap
metadata:
    creationTimestamp: null
    name: iam-cm
    namespace: <namespace>
```
Apply file deploy

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iam-svc
  namespace: <namespace>
spec:
  type: NodePort
  selector:
    app: iam
  ports:
    -  port: 8080 # listen on this port
       targetPort: 8080 # forward traffic to this port of containers
       nodePort: 30007
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iam
  labels:
    app: iam
  namespace: <namespace>
spec:
  selector:
    matchLabels:
      app: iam
  replicas: 1
  revisionHistoryLimit: 3
  strategy:
    rollingUpdate:
      maxSurge: 1         # how many pods we can add at a time
      maxUnavailable: 1   # maxUnavailable define how many pods can be unavailable
    type: RollingUpdate   # during the rolling update
  template:
    metadata:
      namespace: chatbot
      labels:
        app: iam
        img_tag: 'v3.1'
    spec:
      imagePullSecrets:
        - name: pullimagesecret
      containers:
        - name: iam
          image: quay.io/keycloak/keycloak:26.1.2
          imagePullPolicy: 'Always'
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 4
              memory: 8Gi
          envFrom:
            - configMapRef:
                name: iam-cm
          ports:
          - name: http
            containerPort: 8080
```
Sau khi apply xong deployment thì service sẽ được expose Nodeport  
Nếu deployment không sử dụng node port thì apply file 
```yaml
# k apply -f ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: chatbot-dev
  name: iam-ing
  annotations:
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-body-size: "200m"
    nginx.ingress.kubernetes.io/use-forwarded-headers: "true"
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/app-root: /auth/realms/<client-id>/account
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/proxy-buffer-size: 8k

spec:
  rules:
    - host: keycloak.ai
      http:
        paths:
          - path: /auth
            pathType: Prefix
            backend:
              service:
                name: iam-svc
                port:
                  number: 8080
```
<h2 style="color:orange">3. Cài nginx trên front-proxy</h2>
Trên server front-proxy

```
# vim /etc/nginx/conf.d/fptai.conf
add các dòng

upstream iam {
    server 10.86.0.6:30007;  # ip các node worker mà k8s tạo pod keycloak
    server 10.86.0.7:30007;
    server 10.86.0.8:30007;
}

server {
    server_name keycloak.ai;
    access_log /var/log/nginx/keycloak.ai.access.log;
    error_log /var/log/nginx/keycloak.ai.error.log;

    charset utf-8;
    client_max_body_size 100M;

    location / {
        # include includes/k8s-proxy.conf;

        proxy_pass http://iam;

        proxy_pass_request_headers      on;
        proxy_set_header                Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-For $remote_addr;
        proxy_set_header  X-Forwarded-Host $host;


        location = / {
            return 301 /auth/realms/<client-id>/account/;
        }
    }
}
```
