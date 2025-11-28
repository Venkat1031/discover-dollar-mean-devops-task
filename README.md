In this DevOps task, you need to build and deploy a full-stack CRUD application using the MEAN stack (MongoDB, Express, Angular 15, and Node.js). The backend will be developed with Node.js and Express to provide REST APIs, connecting to a MongoDB database. The frontend will be an Angular application utilizing HTTPClient for communication.  

The application will manage a collection of tutorials, where each tutorial includes an ID, title, description, and published status. Users will be able to create, retrieve, update, and delete tutorials. Additionally, a search box will allow users to find tutorials by title.

## Project setup

### Node.js Server

cd backend

npm install

You can update the MongoDB credentials by modifying the `db.config.js` file located in `app/config/`.

Run `node server.js`

### Angular Client

cd frontend

npm install

Run `ng serve --port 8081`

You can modify the `src/app/services/tutorial.service.ts` file to adjust how the frontend interacts with the backend.

Navigate to `http://localhost:8081/`
PART 2 — DevOps Implementation (Required for Assignment)
---------------------------------------------------------

Below are the steps I completed to containerize, deploy, and automate the application.

## 1. Dockerization
Backend Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]

Frontend Dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=build /app/dist/* /usr/share/nginx/html
EXPOSE 80

## 2. Docker Compose Setup

docker-compose.yml:

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

## 3. Nginx Reverse Proxy

nginx/default.conf:

server {
    listen 80;

    location /api/ {
        proxy_pass http://backend:3000/;
    }

    location / {
        proxy_pass http://frontend:80/;
    }
}


Now the entire app loads on:

👉 http://<EC2-PUBLIC-IP>

## 4. AWS EC2 Deployment
EC2 Setup

Ubuntu 22.04 LTS

t2.micro

Security group ports opened: 22, 80, 3000, 4200

Install Docker
sudo apt update -y
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker

Clone the repo
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

Deploy
sudo docker-compose up -d


Application URL:

👉 http://13.233.83.36/

## 5. GitHub Actions CI/CD Pipeline

This pipeline automatically:

✔ Builds Docker images
✔ Pushes them to Docker Hub
✔ SSHs into EC2
✔ Pulls updated images
✔ Restarts containers

.github/workflows/ci-cd.yml:

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
          cd <repo-name>
          git pull
          sudo docker-compose pull
          sudo docker-compose up -d --force-recreate

## 6. Required Screenshots

Create a folder /screenshots and add:

GitHub repo structure

Docker build images

Docker Hub images

docker ps output

EC2 instance running

Security group ports

Application running on EC2 IP

GitHub Actions pipeline success

Nginx config

Logs showing successful update

## 7. Final URLs to Provide

GitHub repo
👉

Docker Hub
👉
EC2 Live URL 
👉http://13.233.83.36/
