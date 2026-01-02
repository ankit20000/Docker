# DockerLAB 1 — Backend (Python API)
🎯 Goal

Create a backend API container that returns data.

📄 backend/app.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Backend API"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

📄 backend/Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY app.py .

RUN pip install flask

EXPOSE 5000

CMD ["python", "app.py"]

🧪 Hands-on Steps (Tell Students)
cd backend
docker build -t backend-api .
docker run -d -p 5000:5000 backend-api


✅ Open browser → http://localhost:5000

🧠 Trainer Notes

Explain EXPOSE vs -p

Show docker ps and docker logs

🔹 LAB 2 — Frontend (NGINX)
🎯 Goal

Serve frontend via NGINX container

📄 frontend/index.html
<!DOCTYPE html>
<html>
<head>
  <title>Docker Lab</title>
</head>
<body>
  <h1>Frontend Container</h1>
  <p>Backend running separately</p>
</body>
</html>

📄 frontend/nginx.conf
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}

📄 frontend/Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

🧪 Hands-on Steps
cd frontend
docker build -t frontend-ui .
docker run -d -p 8080:80 frontend-ui


✅ Open → http://localhost:8080

🧠 Trainer Notes

Explain reverse proxy will come later

NGINX is used in almost every production system

🔹 LAB 3 — Docker Network (Frontend ↔ Backend)
🎯 Goal

Enable container-to-container communication

Step 1: Create network
docker network create app-network

Step 2: Run backend on network
docker run -d \
--name backend \
--network app-network \
backend-api

Step 3: Run frontend on same network
docker run -d \
--name frontend \
--network app-network \
-p 8080:80 \
frontend-ui

🧠 Trainer Notes

Containers talk using names

No IP needed

This is Kubernetes service concept base

🔹 LAB 4 — Database with Volume (MySQL)
🎯 Goal

Persist DB data using Docker volume

Step 1: Create volume
docker volume create mysql-data

Step 2: Run MySQL container
docker run -d \
--name mysql \
--network app-network \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=appdb \
mysql:8

Step 3: Verify persistence
docker stop mysql
docker rm mysql
docker run -d \
--name mysql \
--network app-network \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
mysql:8


➡️ Data still exists ✅

🧠 Trainer Notes

🔥 THIS IS REAL PRODUCTION PAIN
Explain:

“Without volume → DB data lost”

🔹 LAB 5 — Bind Mount (NGINX Live Reload)
🎯 Goal

Show bind mount for development

docker run -d \
-p 8081:80 \
-v $(pwd)/frontend:/usr/share/nginx/html \
nginx


Change index.html → auto reflect in browser.

🧠 Trainer Notes

Bind mount = local dev

Volume = production

🔹 LAB 6 — Resource Limits
docker run -d \
--memory="256m" \
--cpus="0.5" \
nginx

🧠 Trainer Notes

One bad container should not kill server

Used heavily in prod & CI runners

🔹 LAB 7 — Cleanup (VERY IMPORTANT)
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker image prune -f
docker volume prune -f

🧠 Trainer Notes

CI servers crash due to disk full

Cleanup is real DevOps skill

🔹 FINAL LAB ARCHITECTURE (SHOW THIS)
Browser
   ↓
NGINX Frontend
   ↓
Backend API
   ↓
MySQL + Volume
