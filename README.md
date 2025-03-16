# github_actions
# Full-Stack Application with React, Flask, Docker, Kubernetes, and CI/CD

This project is a full-stack application built with a React frontend and a Flask backend, containerized using Docker, and deployed to Kubernetes. It also includes a CI/CD pipeline with GitHub Actions for continuous integration and deployment.

## Requirements
- Git
- Docker
- Node.js (for React)
- Python 3.9+ (for Flask)
- Kubernetes (kubectl)
- Minikube (optional, for local Kubernetes setup)

## Setup

Create the following directory structure:


### Frontend (React)
1. Create a React app inside the `frontend` folder:

```bash
cd frontend
npx create-react-app .


## Step 1: Set Up Project Structure

1. In your terminal, create a new project directory and navigate into it:
    ```
    mkdir fullstack-app && cd fullstack-app
    ```

2. Create the necessary folders and files:
    ```
    mkdir frontend backend k8s
    touch frontend/Dockerfile backend/Dockerfile k8s/deployment.yaml k8s/service.yaml
    ```

## Step 2: Set Up Frontend (React)

1. Navigate to the `frontend/` directory and create a React app:
    ```
    cd frontend
    npx create-react-app .
    ```

2. Replace the contents of `src/App.js` with the following code:

    ```javascript
    import React, { useState, useEffect } from "react";

    function App() {
      const [message, setMessage] = useState("");

      useEffect(() => {
        fetch("/api")
          .then((res) => res.json())
          .then((data) => setMessage(data.message));
      }, []);

      return (
        <div>
          <h1>React + Flask App</h1>
          <p>Backend says: {message}</p>
        </div>
      );
    }

    export default App;
    ```

3. Create a `Dockerfile` for the frontend in `frontend/Dockerfile`:
    ```dockerfile
    FROM node:18-alpine
    WORKDIR /app
    COPY package.json package-lock.json ./ 
    RUN npm install
    COPY . . 
    RUN npm run build
    CMD ["npm", "start"]
    EXPOSE 3000
    ```

## Step 3: Set Up Backend (Flask)

1. Navigate to the `backend/` directory and create `app.py` and `requirements.txt`:
    ```
    cd ../backend
    touch app.py requirements.txt
    ```

2. Add the following code to `app.py`:
    ```python
    from flask import Flask, jsonify

    app = Flask(__name__)

    @app.route("/api")
    def home():
        return jsonify({"message": "Hello from Flask Backend!"})

    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)
    ```

3. Add the necessary dependencies to `requirements.txt`:
    ```
    flask
    ```

4. Create a `Dockerfile` for the backend in `backend/Dockerfile`:
    ```dockerfile
    FROM python:3.9
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    COPY . .
    CMD ["python", "app.py"]
    EXPOSE 5000
    ```

## Step 4: Build and Run Docker Containers Locally

1. Navigate back to the root project folder:
    ```
    cd ..
    ```

2. Build the frontend and backend Docker images:
    ```
    docker build -t frontend:latest ./frontend
    docker build -t backend:latest ./backend
    ```

3. Run the containers:
    ```
    docker run -d -p 3000:3000 frontend:latest
    docker run -d -p 5000:5000 backend:latest
    ```

4. Test if it's working by opening `http://localhost:3000` in your browser. You should see "Backend says: Hello from Flask Backend!"

## Step 5: Set Up Kubernetes Deployment

1. Define the Kubernetes Deployment in `k8s/deployment.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webapp
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: webapp
      template:
        metadata:
          labels:
            app: webapp
        spec:
          containers:
            - name: frontend
              image: your-dockerhub-username/frontend:latest
              ports:
                - containerPort: 3000
            - name: backend
              image: your-dockerhub-username/backend:latest
              ports:
                - containerPort: 5000
    ```

2. Define the Kubernetes Service in `k8s/service.yaml`:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: webapp
    spec:
      type: LoadBalancer
      selector:
        app: webapp
      ports:
        - protocol: TCP
          port: 80
          targetPort: 3000
    ```

3. Apply the Kubernetes configurations:
    ```
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    ```

4. Get the external IP by running:
    ```
    kubectl get services
    ```

5. Open the EXTERNAL-IP in your browser.

## Step 6: Set Up GitHub Actions CI/CD

1. Add the following secrets to GitHub:
    - `DOCKER_USERNAME` → Your Docker Hub username
    - `DOCKER_PASSWORD` → Your Docker Hub password

2. Create the `.github/workflows/ci.yml` file:
    ```
    mkdir -p .github/workflows
    touch .github/workflows/ci.yml
    ```

3. Add the CI/CD pipeline in `.github/workflows/ci.yml`:
    ```yaml
    name: Full-Stack CI/CD Pipeline

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main

    jobs:
      build-frontend:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout Code
            uses: actions/checkout@v3
          - name: Set up Node.js
            uses: actions/setup-node@v3
            with:
              node-version: '18'
          - name: Install Dependencies & Build
            run: |
              cd frontend
              npm install
              npm run build
          - name: Upload Artifact
            uses: actions/upload-artifact@v3
            with:
              name: frontend-build
              path: frontend/build

      build-backend:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout Code
            uses: actions/checkout@v3
          - name: Set up Python
            uses: actions/setup-python@v4
            with:
              python-version: '3.9'
          - name: Install Dependencies & Test
            run: |
              cd backend
              pip install -r requirements.txt
              pytest test_app.py
          - name: Upload Backend Artifact
            uses: actions/upload-artifact@v3
            with:
              name: backend-code
              path: backend

      dockerize:
        runs-on: ubuntu-latest
        needs: [build-frontend, build-backend]
        steps:
          - name: Checkout Code
            uses: actions/checkout@v3
          - name: Login to Docker Hub
            run: |
              echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          - name: Build & Push Images
            run: |
              docker build -t ${{ secrets.DOCKER_USERNAME }}/frontend:latest ./frontend
              docker push ${{ secrets.DOCKER_USERNAME }}/frontend:latest
              docker build -t ${{ secrets.DOCKER_USERNAME }}/backend:latest ./backend
              docker push ${{ secrets.DOCKER_USERNAME }}/backend:latest
    ```

## Step 7: Final Steps

1. Commit and push the code to GitHub:
    ```
    git add .
    git commit -m "Initial Full-Stack CI/CD"
    git push origin main
    ```

2. Go to GitHub → Actions to see the pipeline running.

---

Now you have a full-stack React + Flask application with Docker, Kubernetes, and a CI/CD pipeline using GitHub Actions. You can modify this as needed for your specific requirements.
