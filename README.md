
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
