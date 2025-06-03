# Dockerized Flask App

This is a simple "Hello World" Python Flask application, containerized using Docker. It runs on port `5000` and responds with a message when accessed via the root URL (`/`).

## Requirements

- Docker installed.
- Git (optional, if cloning from a repo)
- Python 3 (for local testing without Docker)

## 1. Project Structure

    docker-flask-app/
    ├── app.py
    ├── requirements.txt
    ├── Dockerfile
    └── .github/
        └── workflows/
            └── deploy.yml



### 1.1 Create App Folder

    #mkdir ~/docker-flask-app
    #cd ~/docker-flask-app

### 1.2 Create app.py

    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def home():
        return "Hello from Flask in Docker!"
    
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=5000)

### 1.3 Create requirements.txt

    flask

### 1.4 Create Dockerfile

    FROM python:3.10-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    COPY app.py .
    EXPOSE 5000
    CMD ["python", "app.py"]

### 1.5 Create GitHub Actions CI/CD – .github/workflows/deploy.yml

    name: Deploy Flask Docker App to EC2
    
    on:
      push:
        branches: [ main ]
    
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
    
        steps:
        - name: Checkout Code
          uses: actions/checkout@v2
    
        - name: Set up Docker
          uses: docker/setup-buildx-action@v2
    
        - name: Login to DockerHub
          run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
    
        - name: Build & Push Docker image
          run: |
            docker build -t flask-docker-app .
            docker tag flask-docker-app ${{ secrets.DOCKER_USERNAME }}/flask-docker-app:latest
            docker push ${{ secrets.DOCKER_USERNAME }}/flask-docker-app:latest
    
        - name: SSH Deploy to EC2
          uses: appleboy/ssh-action@master
          with:
            host: ${{ secrets.EC2_HOST }}
            username: ${{ secrets.EC2_USER }}
            key: ${{ secrets.EC2_KEY }}
            script: |
              docker pull ${{ secrets.DOCKER_USERNAME }}/flask-docker-app:latest
              docker stop flask-container || true
              docker rm flask-container || true
              docker run -d --name flask-container -p 5000:5000 ${{ secrets.DOCKER_USERNAME }}/flask-docker-app:latest


### 1.6 Create GitHub Secrets

**In your GitHub repo:** Go to Settings > Secrets and Variables > Actions and add these secrets:

| Name              | Description                              |
| ----------------- | ---------------------------------------- |
| `EC2_HOST`        | Public IP of EC2                         |
| `EC2_USER`        | Usually `ubuntu`                         |
| `EC2_KEY`         | Your private SSH key (paste raw content) |
| `DOCKER_USERNAME` | Docker Hub username                      |
| `DOCKER_PASSWORD` | Docker Hub password                      |


## 2. Build Docker Image

    #docker build -t flask-hello .

## 3. Run Docker Container

    #docker run -d -p 5000:5000 flask-hello

## 4. Access the App

Open a browser or run:
    #curl http://localhost:5000

**Expected output:**
Hello from Flask in Docker!

## Development Without Docker (Optional)

You can also run the app locally for testing without Docker:

    #pip install flask
    #python app.py
    
    Then access via: http://localhost:5000

## Stopping & Cleaning Up
Stop all running containers:

    #docker ps
    #docker stop <container_id>

**Remove image:**

    #docker rmi flask-hello
