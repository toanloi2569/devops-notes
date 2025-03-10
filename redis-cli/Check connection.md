# Check redis connection

### Using python
```
from redis import Redis

REDIS_HOST = 'localhost'
REDIS_PORT = 6379
REDIS_PASSWORD = 'password'

r = Redis(host=REDIS_HOST,port=REDIS_PORT,db=0,password=REDIS_PASSWORD)
r.ping()
```

### Using redis cli
**Install redis-cli**
```
apt-get install redis-tools
```

**Ping**
```
redis-cli -h XXX.XXX.XXX.XXX -p YYYY -a 'password'

PING
```