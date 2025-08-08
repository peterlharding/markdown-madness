# Connecting Docker Container to Host Services

## Problem
Running Apache Superset in a Docker container and need to connect to a PostgreSQL database accessible via SSH tunnel on `127.0.0.1:15432`.

## Solution
Use special hostnames to map localhost addresses into the Docker network.

## Hostnames by Operating System

### Docker Desktop (Windows/Mac)
```
host.docker.internal:15432
```

### Linux
**Option 1:** Use Docker bridge gateway
```
172.17.0.1:15432
```

**Option 2:** Add host mapping to docker run command
```bash
docker run --add-host=host.docker.internal:host-gateway [other options]
```
Then use: `host.docker.internal:15432`

### Docker Compose
Add to your service definition:
```yaml
services:
  superset:
    # ... other configuration
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
Then use: `host.docker.internal:15432`

## Superset Database Configuration
- **Host**: `host.docker.internal` (or `172.17.0.1` on Linux)
- **Port**: `15432`
- **Database**: your postgres database name
- **Username/Password**: your postgres credentials

## Key Point
`localhost` and `127.0.0.1` inside a container refer to the container itself, not the host machine. These special hostnames bridge the container network to your host network.