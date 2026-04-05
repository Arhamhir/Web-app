# TaskFlow - Part I Deployment Guide

This repository contains a simple full-stack web application for the assignment.

- Frontend: React + Vite
- Backend: FastAPI
- Database: PostgreSQL

The Part I requirement is met like this:
1. Write Dockerfiles for the web app.
2. Build Docker images.
3. Push images to Docker Hub.
4. Use docker-compose to launch the app on EC2.
5. Attach a persistent volume to PostgreSQL.

## Files used for Part I

- [backend/Dockerfile](backend/Dockerfile)
- [frontend/Dockerfile](frontend/Dockerfile)
- [docker-compose.yml](docker-compose.yml)

## 1) What you do on EC2 after cloning the repo

### Install Docker and Git
```bash
sudo apt update
sudo apt install -y git
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
docker --version
docker compose version
```

### Clone the repo
```bash
git clone https://github.com/Arhamhir/Web-app.git
cd Web-app
```

### Set variables for your deployment
```bash
cp .env.example .env
```

Then load the environment variables:
```bash
export DOCKERHUB_USERNAME=your_dockerhub_username
export PUBLIC_IP=3.239.92.83
export POSTGRES_PASSWORD=taskflow123
export JWT_SECRET_KEY='your_strong_secret_here'
```

Explanation:
- `DOCKERHUB_USERNAME` is used by docker-compose to pull your published images.
- `PUBLIC_IP` is used when building the frontend image so the browser calls the EC2 backend.
- `POSTGRES_PASSWORD` protects the database container.
- `JWT_SECRET_KEY` signs login tokens.

## 2) Build and push images to Docker Hub

### Login to Docker Hub
```bash
docker login
```

### Build backend image
```bash
docker build -t $DOCKERHUB_USERNAME/taskflow-backend:part1 ./backend
```

### Build frontend image
```bash
docker build --build-arg VITE_API_URL=http://$PUBLIC_IP:8000 -t $DOCKERHUB_USERNAME/taskflow-frontend:part1 ./frontend
```

### Push both images
```bash
docker push $DOCKERHUB_USERNAME/taskflow-backend:part1
docker push $DOCKERHUB_USERNAME/taskflow-frontend:part1
```

Explanation:
- The backend image contains the FastAPI server and its code.
- The frontend image contains the built React app.
- Both images are published to Docker Hub so the deployment can be reproduced.

## 3) Launch the app with docker-compose

The root [docker-compose.yml](docker-compose.yml) uses your Docker Hub images and creates a PostgreSQL container with persistent storage.

### Start the stack
```bash
docker compose pull
docker compose up -d
```

### Check containers
```bash
docker compose ps
```

### Check logs if needed
```bash
docker compose logs --tail=100 db
docker compose logs --tail=100 backend
docker compose logs --tail=100 frontend
```

Explanation:
- `db` is PostgreSQL.
- `backend` is the FastAPI container.
- `frontend` is the React app served by Nginx.
- PostgreSQL data stays persistent in the Docker volume `postgres_data`.

## 4) URLs to verify

Use these in your browser:
- Frontend: `http://3.239.92.83`
- Backend API: `http://3.239.92.83:8000`
- Swagger docs: `http://3.239.92.83:8000/docs`

## 5) Quick checks

```bash
curl http://localhost:8000/
curl -I http://localhost
```

Expected:
- Backend returns: `{"message":"Welcome to TaskFlow API"}`
- Frontend returns HTTP 200

## 6) What the compose file does

The compose file:
- pulls the backend image from Docker Hub
- pulls the frontend image from Docker Hub
- starts PostgreSQL in a separate container
- mounts a persistent volume for PostgreSQL data
- exposes frontend on port 80 and backend on port 8000

## 7) Useful stop commands

```bash
docker compose down
```

To stop and remove the database volume too:
```bash
docker compose down -v
```

## 8) Notes for submission

For the report, include screenshots of:
1. Docker build commands
2. Docker push to Docker Hub
3. `docker compose ps`
4. Frontend opened in browser
5. Backend docs opened in browser
6. PostgreSQL volume visible in the compose file

That is enough for Part I.
