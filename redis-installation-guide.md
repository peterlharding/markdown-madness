
# Redis Client Installation on Amazon Linux 2023

## 🚨 Problem
The default Amazon Linux 2023 repositories don't include the Redis package:
```bash
$ dnf install redis
No match for argument: redis
Error: Unable to find a match: redis
```

## 🔧 Solution Methods

### Method 1: Enable EPEL Repository (Recommended)
```bash
# Enable EPEL repository
sudo dnf install epel-release -y

# Update package cache
sudo dnf update -y

# Install Redis
sudo dnf install redis -y

# Verify installation
redis-cli --version
```

### Method 2: Search for Available Packages
```bash
# Search for Redis-related packages
dnf search redis

# List available packages with redis in the name
dnf list available | grep -i redis
```

### Method 3: Install from Source
```bash
# Install build dependencies
sudo dnf groupinstall "Development Tools" -y
sudo dnf install wget gcc make -y

# Download Redis source
cd /tmp
wget https://download.redis.io/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
cd redis-stable

# Compile Redis
make

# Install Redis binaries
sudo make install

# Verify installation
redis-cli --version
```

### Method 4: Use Docker
```bash
# Install Docker
sudo dnf install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Run Redis container
docker run -d --name redis-server -p 6379:6379 redis:latest

# Use Redis CLI from container
docker exec -it redis-server redis-cli
```

### Method 5: Install via Snap
```bash
# Install snapd
sudo dnf install snapd -y
sudo systemctl enable --now snapd.socket

# Install Redis via snap
sudo snap install redis
```

## ✅ Testing Redis Client

### Basic Connection Tests
```bash
# Test connection to Redis server
redis-cli -h your-redis-host -p 6379 ping

# Connect to local Redis
redis-cli ping

# Connect with authentication
redis-cli -h your-redis-host -p 6379 -a your-password ping

# Interactive mode
redis-cli -h your-redis-host -p 6379
```

### Common Redis Commands
```bash
# Set a key-value pair
redis-cli set mykey "Hello World"

# Get a value
redis-cli get mykey

# List all keys
redis-cli keys "*"

# Monitor Redis commands in real-time
redis-cli monitor

# Get Redis info
redis-cli info
```

## 🎯 For Superset Integration

If setting up Redis for Apache Superset:

```bash
# Install Python Redis client
python3.11 -m pip install redis

# Test Python Redis connection
python3.11 -c "import redis; r = redis.Redis(host='localhost', port=6379); print(r.ping())"

# Install additional Python packages for Superset
python3.11 -m pip install celery redis hiredis
```

## 🔍 Verification Commands

```bash
# Check Redis client version
redis-cli --version

# Check if Redis tools are available
which redis-cli
which redis-server

# Test basic functionality
redis-cli --help
```

## 📋 Troubleshooting

### If EPEL doesn't work:
```bash
# Try enabling Amazon Linux Extras
sudo amazon-linux-extras list | grep redis
sudo amazon-linux-extras install redis -y
```

### If compilation fails:
```bash
# Install additional dependencies
sudo dnf install tcl-devel -y
```

### Check available repositories:
```bash
# List enabled repositories
dnf repolist

# Search across all repositories
dnf search redis --showduplicates
```

## 🎯 Recommended Approach

1. **Try Method 1** (EPEL) first - cleanest package management
2. **Fall back to Method 3** (source compilation) - most reliable
3. **Use Method 4** (Docker) if you need Redis server as well

---

**Note**: Method 1 (EPEL) is usually the best approach for production systems as it provides proper package management and security updates.


Final Note:  Needed to 'dnf install redis6'

There is no 'redis' or 'redis7' for al2023



