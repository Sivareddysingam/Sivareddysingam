
Step 1: Dockerization
Clone the Repository: Clone the Wisecow application repository.

bash
Copy code
git clone https://github.com/nyrahul/wisecow.git
cd wisecow
Create a Dockerfile: Develop a Dockerfile in the root of the project.

Dockerfile
Copy code
# Use a base image
FROM python:3.9-slim

# Set environment variables
ENV PYTHONUNBUFFERED 1

# Set the working directory
WORKDIR /app

# Copy the requirements file and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the application port
EXPOSE 8000

# Run the application
CMD ["python", "app.py"]
Build and Test the Docker Image:

bash
Copy code
docker build -t wisecow-app .
docker run -p 8000:8000 wisecow-app
Step 2: Kubernetes Deployment
Create Kubernetes Manifests:

Deployment: Create a deployment.yaml file.

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: <your-container-registry>/wisecow-app:latest
        ports:
        - containerPort: 8000
Service: Create a service.yaml file.

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer
Deploy to Kubernetes:

bash
Copy code
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Step 3: Continuous Integration and Deployment (CI/CD)
Setup GitHub Actions Workflow:
Create a .github/workflows/ci-cd.yml file.
yaml
Copy code
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Login to DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/wisecow-app:latest .
        docker push ${{ secrets.DOCKER_USERNAME }}/wisecow-app:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up kubectl
      uses: azure/setup-kubectl@v1
      with:
        version: 'latest'

    - name: Deploy to Kubernetes
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}
      run: |
        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml
Step 4: TLS Implementation
Create and Configure TLS Certificates:

Use a service like Let's Encrypt or self-signed certificates.
Create Kubernetes secrets to store the TLS certificates.
bash
Copy code
kubectl create secret tls wisecow-tls --cert=path/to/tls.crt --key=path/to/tls.key
Update Kubernetes Manifests for TLS:

Ingress: Create an ingress.yaml file.
yaml
Copy code
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - wisecow.example.com
    secretName: wisecow-tls
  rules:
  - host: wisecow.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-service
            port:
              number: 80
Deploy the Ingress:

bash
Copy code
kubectl apply -f ingress.yaml
Expected Artifacts
Repository Contents:
Wisecow application source code.
Dockerfile for the application.
Kubernetes manifests (deployment.yaml, service.yaml, ingress.yaml).
GitHub Actions workflow file (ci-cd.yml).
Access Control
Set Repository to Public: Ensure the GitHub repository is public.


ANS 2 :

System Health Monitoring Script
Application Health Checker
System Health Monitoring Script
Here's a Python script to monitor the health of a Linux system by checking CPU usage, memory usage, disk space, and running processes. If any of these metrics exceed predefined thresholds, it will send an alert to a log file.

python
Copy code
import psutil
import logging
import datetime

# Set up logging
logging.basicConfig(filename='system_health.log', level=logging.INFO)

# Define thresholds
CPU_THRESHOLD = 80
MEMORY_THRESHOLD = 80
DISK_THRESHOLD = 80

def check_cpu():
    usage = psutil.cpu_percent(interval=1)
    if usage > CPU_THRESHOLD:
        logging.warning(f'{datetime.datetime.now()}: High CPU usage detected: {usage}%')
    return usage

def check_memory():
    memory = psutil.virtual_memory()
    usage = memory.percent
    if usage > MEMORY_THRESHOLD:
        logging.warning(f'{datetime.datetime.now()}: High memory usage detected: {usage}%')
    return usage

def check_disk():
    disk = psutil.disk_usage('/')
    usage = disk.percent
    if usage > DISK_THRESHOLD:
        logging.warning(f'{datetime.datetime.now()}: High disk usage detected: {usage}%')
    return usage

def check_processes():
    processes = [p.info for p in psutil.process_iter(attrs=['pid', 'name', 'username'])]
    logging.info(f'{datetime.datetime.now()}: Running processes: {processes}')
    return processes

def monitor_system():
    cpu = check_cpu()
    memory = check_memory()
    disk = check_disk()
    processes = check_processes()
    logging.info(f'{datetime.datetime.now()}: CPU usage: {cpu}%, Memory usage: {memory}%, Disk usage: {disk}%')

if __name__ == "__main__":
    monitor_system()
Application Health Checker
Here's a Python script that checks the uptime of an application by assessing HTTP status codes. It will log if the application is 'up' or 'down'.

python
Copy code
import requests
import logging
import datetime

# Set up logging
logging.basicConfig(filename='app_health.log', level=logging.INFO)

# Define the application URL and the expected HTTP status code for a healthy application
APP_URL = 'http://example.com'  # Replace with the actual URL
EXPECTED_STATUS_CODE = 200

def check_app_health():
    try:
        response = requests.get(APP_URL)
        if response.status_code == EXPECTED_STATUS_CODE:
            logging.info(f'{datetime.datetime.now()}: Application is UP. Status Code: {response.status_code}')
        else:
            logging.error(f'{datetime.datetime.now()}: Application is DOWN. Status Code: {response.status_code}')
    except requests.exceptions.RequestException as e:
        logging.error(f'{datetime.datetime.now()}: Application is DOWN. Error: {e}')

if __name__ == "__main__":
    check_app_health()
How to Run the Scripts
System Health Monitoring Script:

Save the script as system_health_monitor.py.
Run the script using python3 system_health_monitor.py.
Application Health Checker:

Save the script as app_health_checker.py.
Run the script using python3 app_health_checker.py.
Notes:
Make sure you have the required Python packages installed. You can install them using:
bash
Copy code
pip install psutil requests
Adjust the thresholds and the application URL as per your requirements.
For continuous monitoring, consider running these scripts as cron jobs or using a task scheduler.





