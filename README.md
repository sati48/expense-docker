# Dockerized Expense Tracking Application

## Development Journey and Setup Guide

### Initial Setup on EC2
```bash
# Install Docker
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Building and Troubleshooting Process

1. Initial Stack Deployment
```bash
# Build and start all containers
docker-compose up -d --build

# Check container status
docker-compose ps
```

2. Database Setup Verification
```bash
# Check MySQL logs for initialization
docker-compose logs mysql

# Verify MySQL is healthy
docker ps  # Look for (healthy) status
```

3. Backend Service Setup
```bash
# Check backend logs
docker-compose logs backend

# Test backend health endpoint
curl http://localhost:8080/health
```

4. Frontend Setup and Fixes
```bash
# Check frontend logs
docker-compose logs frontend

# Rebuild frontend after nginx config changes
docker-compose stop frontend
docker-compose rm -f frontend
docker-compose up -d --build frontend
```

### Issues Faced and Solutions

1. Docker Permission Issues
- Problem: "permission denied while trying to connect to the Docker daemon socket"
```bash
docker images
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
```
- Solution: Add user to docker group and restart session
```bash
sudo usermod -aG docker ec2-user
# Log out and log back in, or restart the instance
newgrp docker  # To apply group changes without logging out
```

2. Docker Build Platform Incompatibility
- Problem: Architecture mismatch when building images
- Error: "linux/amd64 platform not available"
- Solution: Add platform specification in docker-compose.yml
```yaml
services:
  backend:
    platform: linux/amd64  # Add platform specification
    build:
      context: .
      dockerfile: docker/backend/Dockerfile
```
- Alternative Solution: Use buildx with platform flag
```bash
docker buildx build --platform linux/amd64 .
```

3. Static File Serving Issue
- Problem: Black screen, JavaScript not loading
- Error: Unexpected token '<' in main.bcd36112.js
- Solution: Updated nginx.conf with proper static file configuration
```nginx
location /static/ {
    root /usr/share/nginx/html;
    expires 30d;
    add_header Cache-Control "public, no-transform";
}
```

2. API Connectivity
- Problem: Frontend not connecting to backend
- Solution: Updated nginx proxy configuration
```nginx
location /api/ {
    proxy_pass http://backend:8080/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

3. Network Communication
- Problem: Services not finding each other
- Solution: Verified Docker network setup
```bash
# List networks
docker network ls

# Inspect network
docker network inspect expense-docker_default
```

4. MySQL Initialization
- Problem: Database not initializing properly
- Solution: Added proper volume mapping and initialization script
```yaml
volumes:
  - ./expense-backend-v2/schema/backend.sql:/docker-entrypoint-initdb.d/backend.sql
```

### Useful Debug Commands

```bash
# Check container logs
docker-compose logs [service_name]

# Enter container shell
docker exec -it [container_name] /bin/bash 

# Check container network
docker network inspect [network_name]

# View running containers
docker ps

# Check resource usage
docker stats

# Restart specific service
docker-compose restart [service_name]

# Clean restart
docker-compose down
docker-compose up -d --build
```

This repository contains a containerized 3-tier expense tracking application using Docker and Docker Compose. The application consists of a React frontend, Node.js backend, and MySQL database.

## Project Structure
```
expense-docker/
├── docker/
│   ├── backend/
│   │   └── Dockerfile
│   ├── frontend/
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   └── mysql/
│       └── Dockerfile
├── expense-backend-v2/
│   ├── DbConfig.js
│   ├── index.js
│   ├── TransactionService.js
│   └── schema/
│       └── backend.sql
├── expense-frontend-v2/
│   ├── index.html
│   └── static/
└── docker-compose.yml
```

## Prerequisites
- Docker
- Docker Compose
- Node.js (for local development)
- Git

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/sati48/expense-docker.git
cd expense-docker
```

### 2. Build and Start the Containers
```bash
docker-compose up -d --build
```

This command will:
- Build all service images
- Create a Docker network
- Start all containers in detached mode
- Set up volume for MySQL persistence

### 3. Verify Services
Check if all containers are running:
```bash
docker-compose ps
```

Expected output:
```
NAME                IMAGE                     STATUS
expense-frontend    expense-docker-frontend   Up
expense-backend     expense-docker-backend    Up
expense-mysql       expense-docker-mysql      Up (healthy)
```

### 4. Access the Application
- Frontend: http://localhost (or your server IP)
- Backend API: http://localhost:8080
- MySQL: localhost:3306 (accessible within container network)

## Service Details

### Frontend (Nginx + React)
- Port: 80
- Nginx serves static React files
- Proxies API requests to backend
- Configuration: docker/frontend/nginx.conf

### Backend (Node.js)
- Port: 8080
- REST API for expense management
- Connects to MySQL database
- Health check endpoint: /health

### Database (MySQL)
- Port: 3306
- Database: transactions
- User: expense
- Auto-initialized with schema
- Persistent volume for data storage

## API Endpoints
- `GET /health` - Health check
- `POST /transaction` - Add new transaction
- More endpoints documented in backend code

## Container Network
- Network: expense-docker_default
- Internal container communication using service names
- Frontend → backend via "backend:8080"
- Backend → MySQL via "mysql:3306"

## Monitoring and Logs
View container logs:
```bash
# All services
docker-compose logs

# Specific service
docker-compose logs [service_name]
```

## Persistence
MySQL data is persisted through Docker volumes:
- Volume: mysql-data
- Mount point: /var/lib/mysql

## Security Considerations
- MySQL is accessible only within container network
- Frontend proxies API requests securely
- CORS configured on backend
- Container user permissions set

## Troubleshooting
1. Check container status:
   ```bash
   docker-compose ps
   ```

2. View container logs:
   ```bash
   docker-compose logs [service_name]
   ```

3. Restart services:
   ```bash
   docker-compose restart [service_name]
   ```

4. Rebuild specific service:
   ```bash
   docker-compose up -d --build [service_name]
   ```

## Common Issues and Solutions
1. Black screen in UI
   - Check browser console for JavaScript errors
   - Verify static file serving in Nginx
   - Check frontend container logs

2. Database Connection Issues
   - Ensure MySQL container is healthy
   - Verify database credentials
   - Check network connectivity

3. API Connection Issues
   - Verify backend health check endpoint
   - Check Nginx proxy configuration
   - Verify CORS settings

## Cleanup
To stop and remove all containers:
```bash
docker-compose down
```

To remove all containers and volumes:
```bash
docker-compose down -v
```

## Production Deployment Notes
1. Use environment variables for sensitive data
2. Implement proper logging solution
3. Set up monitoring
4. Use production-grade MySQL configuration
5. Implement SSL/TLS
6. Regular backup strategy
7. Container health checks
8. Resource limits
