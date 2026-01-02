
---

## 🔹 Docker LABS

### 🎯 Goal
Creating frontend and backend application and running those application as containers 

---
LAB 1 — Backend (Python API)
### 📄 backend/app.py

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Backend API"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

### backend/Dockerfile


FROM python:3.10-slim

WORKDIR /app

COPY app.py .

RUN pip install flask

EXPOSE 5000

CMD ["python", "app.py"]


//cd backend
//docker build -t backend-api .
//docker run -d -p 5000:5000 backend-api

---
## 🔹  LAB 2 — Frontend (NGINX)
frontend/index.html
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

---
frontend/nginx.conf
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}

---
frontend/Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

----
Hands-on Steps
cd frontend
docker build -t frontend-ui .
docker run -d -p 8080:80 frontend-ui


✅ Open in browser:
http://localhost:8080

----


🔹 LAB 3 — Docker Network (Frontend ↔ Backend)
🎯 Goal

Enable container-to-container communication using Docker network.

Step 1: Create Network
docker network create app-network

Step 2: Run Backend on Network
docker run -d \
--name backend \
--network app-network \
backend-api

---
LAB 4 — Database with Docker Volume (MySQL)
🎯 Goal

Persist database data using Docker volumes.

Step 1: Create Volume
docker volume create mysql-data

Step 2: Run MySQL Container
docker run -d \
--name mysql \
--network app-network \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=appdb \
mysql:8

Step 3: Verify Persistence
docker stop mysql
docker rm mysql

docker run -d \
--name mysql \
--network app-network \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
mysql:8


➡️ Database data is preserved ✅

🔥 Real production pain point
Without volumes → database data is lost
---------
🔹 LAB 5 — Bind Mount (NGINX Live Reload)
🎯 Goal

Demonstrate bind mount for local development.

docker run -d \
-p 8081:80 \
-v $(pwd)/frontend:/usr/share/nginx/html \
nginx


Change index.html → refresh browser → changes appear instantly.

🧠 Trainer Notes

Bind mount → local development

Volume → production workloads

---

🔹 LAB 6 — Cleanup (VERY IMPORTANT)
🎯 Goal

Clean unused Docker resources.

docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker image prune -f
docker volume prune -f




