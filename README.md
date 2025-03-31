# Docker Multi-Stage Build for Node.js Application

## Overview
This project demonstrates the use of **Docker multi-stage builds** to optimize a Node.js application by reducing the final image size. Multi-stage builds allow us to separate the build and runtime environments, leading to **smaller, more efficient** Docker images. Additionally, an **Nginx reverse proxy** is configured to enhance performance and manage incoming traffic.

## Technologies Used
- **Node.js** - Backend application
- **Docker** - Containerization
- **Nginx** - Reverse proxy for better performance and load balancing

## Features
- Uses **multi-stage Docker builds** for an optimized image
- Lightweight **Node.js production image**
- Exposes the application on **port 3000**
- **Nginx reverse proxy** for improved performance and request handling

## Project Structure
```
node-multi-stage/
├── Dockerfile
├── package.json
├── package-lock.json
├── node_modules/
├── server.js
├── index.html
└── nginx/
    └── default.conf
```

## Installation & Setup

### 1️⃣ Clone the Repository
```sh
git clone git@github.com:mehraakshat-ops/docker_multi_stage_build.git
cd docker_multi_stage_build
```

### 2️⃣ Build the Docker Image
```sh
docker build -t node-multi-stage .
```

### 3️⃣ Run the Container
```sh
docker run -p 3000:3000 node-multi-stage
```

### 4️⃣ Access the Application
Open your browser and go to:
```
http://localhost:3000
```
If running on AWS EC2, replace `localhost` with your **EC2 public IP**.

## Dockerfile Explanation
```Dockerfile
# Stage 1: Build
FROM node:14 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Stage 2: Run
FROM node:14-slim
WORKDIR /app
COPY --from=build-stage /app .
EXPOSE 3000
ENV NODE_ENV=production
CMD ["node", "server.js"]
```
### Breakdown:
- **Stage 1**: Installs dependencies and prepares the app.
- **Stage 2**: Copies only the necessary files to a **lighter base image**.
- **Result**: A smaller, optimized Docker image.

## Nginx Reverse Proxy Configuration
To enhance performance, an **Nginx reverse proxy** is used to forward requests to the Node.js server. The configuration file (`nginx/default.conf`) should look like this:

```nginx
server {
    listen 80;
    server_name your-ec2-public-ip;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Running Nginx with Docker
1. **Create an Nginx container**:
   ```sh
   docker run -d -p 80:80 -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf nginx
   ```
2. **Restart the Node.js container** and ensure it's running.
3. **Access the app via your EC2 public IP**:
   ```
   http://your-ec2-public-ip/
   ```

## Checking Docker Image Size
To verify the image size:
```sh
docker images | grep node-multi-stage
```

## Deploying on AWS EC2
1. **Launch an EC2 Instance** (Amazon Linux 2 / Ubuntu)
2. **Install Docker**:
   ```sh
   sudo yum install docker -y
   sudo systemctl start docker
   sudo usermod -aG docker ec2-user
   ```
3. **Clone the repo & build the image**:
   ```sh
   git clone git@github.com:mehraakshat-ops/docker_multi_stage_build.git
   cd docker_multi_stage_build
   docker build -t node-multi-stage .
   ```
4. **Run the Node.js container**:
   ```sh
   docker run -d -p 3000:3000 node-multi-stage
   ```
5. **Run the Nginx reverse proxy**:
   ```sh
   docker run -d -p 80:80 -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf nginx
   ```
6. **Access via EC2 Public IP**:
   ```
   http://your-ec2-public-ip/
   ```

## Contributing
Feel free to submit issues or pull requests!

