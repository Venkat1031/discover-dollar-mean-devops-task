🚀 Discover Dollar Mean – DevOps Assignment (MEAN Stack CRUD App)

This project demonstrates the end-to-end DevOps implementation of a full-stack MEAN (MongoDB, Express, Angular 15, Node.js) CRUD application. The application is fully containerized using Docker, deployed on AWS EC2, exposed via Nginx reverse proxy, and automated using a GitHub Actions CI/CD pipeline.

Live Application URL:
👉 http://13.235.95.244

📌 Project Overview

The application manages a collection of tutorials with the following fields:

ID

Title

Description

Published Status

✅ Features

Create, Read, Update, Delete (CRUD) tutorials

Search tutorials by title

RESTful API using Node.js & Express

Frontend using Angular 15 + HttpClient

Database using MongoDB

Fully containerized and deployed on AWS

Automated CI/CD with GitHub Actions

🧰 Tech Stack
Layer	Technology
Frontend	Angular 15, TypeScript
Backend	Node.js, Express.js
Database	MongoDB
DevOps	Docker, Docker Compose
Web Server	Nginx
CI/CD	GitHub Actions
Cloud	AWS EC2 (Ubuntu 22.04)
🏗️ Architecture Flow
Developer Push → GitHub → GitHub Actions CI
→ Build Docker Images
→ Push to Docker Hub
→ SSH to AWS EC2
→ Pull Latest Images
→ Restart Containers using Docker Compose
→ Traffic routed via Nginx Reverse Proxy

⚙️ Local Project Setup
🔹 Backend (Node.js + Express)
cd backend
npm install
node server.js


MongoDB credentials can be modified in:

app/config/db.config.js

🔹 Frontend (Angular)
cd frontend
npm install
ng serve --port 8081


Frontend API interaction file:

src/app/services/tutorial.service.ts


Access locally at:
👉 http://localhost:8081

🐳 Docker Implementation
🔹 Backend Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]

🔹 Frontend Dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=build /app/dist/* /usr/share/nginx/html
EXPOSE 80

📦 Docker Compose Setup
version: '3.9'

services:
  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  backend:
    build: ./backend
    environment:
      - MONGO_URL=mongodb://mongo:27017/mean-crud
    ports:
      - "3000:3000"
    depends_on:
      - mongo

  frontend:
    build: ./frontend
    ports:
      - "4200:80"
    depends_on:
      - backend

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend

volumes:
  mongo_data:

🌐 Nginx Reverse Proxy Configuration
server {
    listen 80;

    location /api/ {
        proxy_pass http://backend:3000/;
    }

    location / {
        proxy_pass http://frontend:80/;
    }
}

☁️ AWS EC2 Deployment
🔹 EC2 Configuration

OS: Ubuntu 22.04 LTS

Instance Type: t2.micro

Opened Ports: 22, 80, 3000, 4200

🔹 Install Docker & Compose
sudo apt update -y
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker

🔹 Deploy Application
git clone https://github.com/venkat1031/discover-dollar-mean-devops-task.git
cd discover-dollar-mean-devops-task
sudo docker-compose up -d


✅ Application accessible at:
👉 http://13.235.95.244

🔄 GitHub Actions CI/CD Pipeline

This pipeline performs:

✅ Docker Image Build
✅ Push to Docker Hub
✅ SSH into AWS EC2
✅ Pull latest images
✅ Restart containers

.github/workflows/ci-cd.yml
name: CI-CD-Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Login DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Backend Image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/mean-backend ./backend

      - name: Build Frontend Image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/mean-frontend ./frontend

      - name: Push Images
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/mean-backend
          docker push ${{ secrets.DOCKER_USERNAME }}/mean-frontend

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.1.0
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ubuntu
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd discover-dollar-mean-devops-task
            git pull
            sudo docker-compose pull
            sudo docker-compose up -d --force-recreate

🔗 Important Links

GitHub Repository:
👉 https://github.com/venkat1031/discover-dollar-mean-devops-task

Docker Hub Images:
Backend 👉 https://hub.docker.com/r/venkat1031/mean-backend

Frontend 👉 https://hub.docker.com/r/venkat1031/mean-frontend

Live Application:
👉 http://13.235.95.244
