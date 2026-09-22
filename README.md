# 📝 Real-Time Collaborative Code Editor

![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-black?style=for-the-badge&logo=socket.io&badgeColor=010101)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)

A high-performance, full-stack collaborative code editor that allows multiple developers to write and edit code simultaneously in real-time. Built with **Yjs** (CRDTs) to ensure conflict-free document synchronization and hosted serverlessly on **AWS ECS Fargate**.

**🔗 Live Demo:** [Application Link](http://docker-aws-yt-alb-3-105354986.ap-south-1.elb.amazonaws.com/)

## Key Features
* **Real-Time Collaboration:** Millisecond-accurate document synchronization across clients.
* **Live Presence & Cursors:** View where other users are typing in real-time.
* **Rich Code Editing:** Integrated **Monaco Editor** (the core of VS Code) for syntax highlighting and intelligent code behavior.
* **Conflict-Free Data Types (CRDT):** Powered by Yjs to guarantee eventual consistency across all distributed clients without locking files.
* **Cloud-Native Deployment:** Containerized with Docker and deployed serverlessly behind an AWS Application Load Balancer.

## Performance Benchmarks (Production)
During load and performance testing on live AWS infrastructure, this application achieved:
* **Sub-90ms Sync Latency:** Maintained highly responsive client-to-client synchronization across active web sockets.
* **High Concurrency:** Successfully supported **30+ simultaneous users** in a single session with zero connection drops or latency spikes.
* **Lean Resource Footprint:** Maintained sub-40ms ALB target response times while consuming **<2% CPU and memory** inside the ECS Fargate container.

## Screenshots

<p align="center">
  <img src="./screenshots/app_screenshot.png" width="45%" alt="Multi-cursor live editing" />
  <br/>
  <b>Live multi-cursor collaborative editing</b>
</p>

<p align="center">
  <img src="./screenshots/ECR.png" width="45%" alt="ECR image pushed" />
  <img src="./screenshots/cluster_and_service_name.png" width="45%" alt="Cluster and Services" />
</p>
<p align="center">
  <b>Docker image pushed to ECR</b>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>ECS Cluster & Service running</b>
</p>

<p align="center">
  <img src="./screenshots/cpu_memory_utilization.png" width="45%" alt="Graph cpu and memory utilization of the service" />
  <img src="./screenshots/target_response_time.png" width="45%" alt="Target response time of ALB" />
</p>
<p align="center">
  <b>CPU & memory utilization (CloudWatch)</b>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>ALB target response time</b>
</p>

## Tech Stack & Architecture

* **Frontend:** React, Monaco Editor, Yjs (y-websocket)
* **Backend:** Node.js, Express, Socket.IO
* **Containerization:** Multi-stage Dockerfile (packaging React static build and Express API into a single image)
* **Cloud & DevOps (AWS):** 
  * Elastic Container Registry (**ECR**) for image hosting
  * Elastic Container Service (**ECS**) using **Fargate** launch type (Serverless compute)
  * Application Load Balancer (**ALB**) for health-checks and traffic routing

## Cloud Architecture Flow
1. **Frontend Assets:** The React build is served statically via the Express backend.
2. **WebSocket Routing:** The AWS ALB is configured to route HTTP traffic and upgrade WebSocket connections.
3. **Container Orchestration:** ECS Fargate scales the Node.js instances seamlessly based on traffic.
4. **State Sync:** Socket.IO and Yjs handle the distribution of state updates (document inserts/deletes) back and forth to connected clients.

---

## Local Development Setup

To run this project locally, you will need **Node.js** and **npm** installed.

### 1. Clone the repository

```bash
git clone https://github.com/YourUsername/collaborative-code-editor.git
cd collaborative-code-editor
```

### 2. Install Dependencies

*(Assuming a standard frontend/backend split — adjust these paths based on your actual folder structure)*

```bash
# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### 3. Run Locally (Without Docker)

You need to run both the frontend React server and the backend Node server.

```bash
# Terminal 1: Start Backend (Port 5000)
cd backend
npm run dev

# Terminal 2: Start Frontend (Port 3000)
cd frontend
npm start
```

### 4. Run Locally (With Docker)

To emulate the production AWS environment locally, you can build and run the multi-stage Docker image:

```bash
# Build the lean production image
docker build -t real-time-editor .

# Run the container locally on port 8080
docker run -p 8080:8080 real-time-editor
```

Navigate to `http://localhost:8080` in your browser.

## 🌐 AWS Deployment Guide

This application is designed to be pushed to AWS ECR and run on ECS Fargate.

### 1. Authenticate Docker with AWS ECR

```bash
aws ecr get-login-password --region <YOUR_REGION> | docker login --username AWS --password-stdin <YOUR_ACCOUNT_ID>.dkr.ecr.<YOUR_REGION>.amazonaws.com
```

### 2. Tag and Push Image

```bash
docker tag real-time-editor:latest <YOUR_ACCOUNT_ID>.dkr.ecr.<YOUR_REGION>.amazonaws.com/real-time-editor:latest
docker push <YOUR_ACCOUNT_ID>.dkr.ecr.<YOUR_REGION>.amazonaws.com/real-time-editor:latest
```

### 3. ECS Deployment

Force a new deployment in your ECS Cluster to pull the latest image, and ensure your ALB Target Group health checks point to the `/health` endpoint on your Express server.

