# Redis Docker Usage Guide

## Connecting to Redis

### If you started Redis with default settings:
```bash
# Connect using redis-cli (if installed locally)
redis-cli -h localhost -p 6379

# Or connect via Docker
docker exec -it <container-name> redis-cli
```

### If you don't know your container name:
```bash
# List running containers
docker ps

# Connect using container ID
docker exec -it <container-id> redis-cli
```

## Basic Redis Commands

Once connected, you can start using Redis:

```redis
# Set and get values
SET mykey "Hello World"
GET mykey

# Work with lists
LPUSH mylist "item1"
LPUSH mylist "item2"
LRANGE mylist 0 -1

# Work with hashes
HSET user:1 name "John" age 30
HGET user:1 name
HGETALL user:1

# Set expiration (TTL)
SET session:123 "user_data"
EXPIRE session:123 3600  # expires in 1 hour

# Check what keys exist
KEYS *
```

## Connecting from Applications

### Python (using redis-py):
```python
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set('foo', 'bar')
value = r.get('foo')
```

### Node.js (using redis package):
```javascript
import redis from 'redis';

const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

await client.connect();
await client.set('key', 'value');
const value = await client.get('key');
```

## Common Docker Redis Configurations

### Basic container:
```bash
docker run -d --name redis-server -p 6379:6379 redis:latest
```

### With persistent data:
```bash
docker run -d --name redis-server -p 6379:6379 -v redis-data:/data redis:latest
```

### With custom config:
```bash
docker run -d --name redis-server -p 6379:6379 -v /path/to/redis.conf:/usr/local/etc/redis/redis.conf redis:latest redis-server /usr/local/etc/redis/redis.conf
```

## Useful Management Commands

```bash
# View Redis logs
docker logs redis-server

# Monitor Redis commands in real-time
docker exec -it redis-server redis-cli monitor

# Get Redis info
docker exec -it redis-server redis-cli info

# Flush all data (be careful!)
docker exec -it redis-server redis-cli flushall
```

## Additional Tips

- Redis runs on port 6379 by default
- Data is stored in memory by default (will be lost when container stops unless you use volumes)
- Use `redis-cli --help` for more command options
- Consider setting up authentication for production use
- Monitor memory usage with `INFO memory` command