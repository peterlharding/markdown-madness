# Apache Superset Setup on AWS EC2 - Complete Guide

This guide walks through setting up Apache Superset from source on an AWS EC2 instance, including troubleshooting common issues.

## Prerequisites

- AWS EC2 instance (t3.large or larger recommended for building frontend)
- SSH or SSM access to the instance
- Git installed on the instance

## Step 1: Initial Setup

### Clone Superset Repository
```bash
git clone https://github.com/apache/superset.git
cd superset
```

### Install Python 3.10+
Superset requires Python 3.10 or higher.

**For Amazon Linux:**
```bash
sudo dnf install -y python3.11 python3.11-pip
```

**For Ubuntu:**
```bash
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-pip
```

## Step 2: Install System Dependencies

### Install Build Tools and Database Libraries
```bash
# For Amazon Linux 2023
sudo dnf install -y gcc gcc-c++ python3-devel pkg-config

# Install MariaDB development libraries (found via: dnf search mysql | grep -i dev)
sudo dnf install -y mariadb1011-devel

# Additional dependencies that may be needed
sudo dnf install -y openssl-devel libffi-devel
```

### Alternative for Ubuntu:
```bash
sudo apt install -y build-essential python3-dev libmysqlclient-dev pkg-config
```

## Step 3: Python Environment Setup

### Create Virtual Environment
```bash
python3.11 -m venv venv
source venv/bin/activate
```

### Install Superset
```bash
# Install Superset in development mode
pip install -e .

# Install development dependencies
pip install -r requirements/development.txt
```

### Common Issues and Solutions

**Issue: mysqlclient compilation errors**
- Solution: Install the MariaDB development packages shown above

**Issue: Click version conflicts**
- Solution: Ensure you're using Python 3.10+ (older Python versions aren't supported)

## Step 4: Database Initialization

### Initialize Database
```bash
# Initialize the database
superset db upgrade

# Create admin user (you'll be prompted for details)
superset fab create-admin

# Initialize roles and permissions
superset init
```

### Optional: Load Example Data
```bash
# Load example datasets (may fail due to various issues)
superset load_examples
```

**Note:** If example loading fails, you can skip this step and start Superset without examples.

## Step 5: Frontend Build (Critical Step)

This is where many setups fail. The frontend assets must be built separately.

### Option A: Build on EC2 (Requires Large Instance)

**Install Node.js:**
```bash
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo dnf install -y nodejs
```

**Build Frontend:**
```bash
cd superset-frontend
npm ci
npm run build
```

**Note:** This requires significant memory and may fail on t3.medium instances.

### Option B: Build on Local Machine (Recommended)

1. Clone Superset on your local machine (Mac/Linux)
2. Install Node.js locally
3. Build the frontend locally:
   ```bash
   cd superset-frontend
   npm ci
   npm run build
   ```
4. Transfer built assets to EC2

### Option C: Add Swap Space for Small EC2 Instances
```bash
# Create 4GB swap file
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Step 6: Transfer Built Assets (If Built Locally)

### On Local Machine:
```bash
# Navigate to main superset directory (not superset-frontend)
cd /path/to/superset

# Verify assets exist
ls -la superset/static/assets/

# Create archive
tar -czf built-assets.tar.gz superset/static/

# Upload to EC2
scp built-assets.tar.gz ec2-user@your-ec2-ip:/tmp/
```

### On EC2 Instance:
```bash
cd /u/src/superset

# Backup existing static directory
mv superset/static superset/static.backup 2>/dev/null || true

# Extract built assets
tar -xzf /tmp/built-assets.tar.gz
```

## Step 7: Start Superset

```bash
# Ensure virtual environment is active
source venv/bin/activate

# Start Superset
superset run -h 0.0.0.0 -p 8088 --with-threads --reload --debugger
```

## Step 8: Configure AWS Security Group

Ensure your EC2 security group allows inbound traffic:
- **Type:** Custom TCP
- **Port:** 8088
- **Source:** 0.0.0.0/0 (or restrict to your IP)

## Step 9: Access Superset

Navigate to: `http://your-ec2-public-ip:8088`

Log in with the admin credentials you created earlier.

## Alternative: Docker Approach

If the above steps prove too complex, consider using Docker:

```bash
# Install Docker
sudo dnf install -y docker
sudo systemctl start docker
sudo usermod -a -G docker ec2-user

# Run Superset
docker run -d -p 8088:8088 -e "SUPERSET_SECRET_KEY=your_secret_key" --name superset apache/superset:latest

# Initialize
docker exec -it superset superset db upgrade
docker exec -it superset superset fab create-admin
docker exec -it superset superset init
```

## Troubleshooting Common Issues

### 404 Errors for Static Assets
- **Cause:** Frontend assets not built
- **Solution:** Complete Step 5 (Frontend Build)

### Memory Issues During Build
- **Cause:** Insufficient memory on EC2 instance
- **Solutions:** 
  - Use larger instance temporarily
  - Add swap space
  - Build locally and transfer assets

### Database Initialization Errors
- **Solution:** Delete `~/.superset/` directory and reinitialize

### Permission Errors
- **Solution:** Ensure proper file permissions and virtual environment activation

## Production Considerations

For production deployments, consider:
- Using PostgreSQL instead of SQLite
- Setting up Nginx as reverse proxy
- Configuring SSL certificates
- Using Gunicorn instead of development server
- Setting up proper logging and monitoring

## Key Files and Directories

- **Main config:** `~/.superset/superset_config.py`
- **Database:** `~/.superset/superset.db` (SQLite)
- **Static assets:** `superset/static/assets/`
- **Logs:** Check console output or configure logging

This guide should help you successfully set up Apache Superset on AWS EC2. The key challenges are usually around Python dependencies, frontend building, and resource constraints on smaller EC2 instances.