<div align="center">

# 🚀 MERN App — CI/CD with GitHub Actions + AWS EC2

### Full-stack MERN application with automated build, test & deploy pipeline to AWS EC2 via SSH

[![MongoDB](https://img.shields.io/badge/MongoDB-Database-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Express](https://img.shields.io/badge/Express.js-Backend-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com/)
[![React](https://img.shields.io/badge/React-Frontend-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org/)
[![Node.js](https://img.shields.io/badge/Node.js-Runtime-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/ec2/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

<br/>

> *Every push to `main` automatically builds Docker images, pushes to Docker Hub, and deploys to AWS EC2 via SSH — zero manual steps.*

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Getting Started](#-getting-started)
- [GitHub Secrets Setup](#-github-secrets-setup)
- [Local Development](#-local-development)
- [Author](#-author)

---

## 📖 Overview

This project demonstrates a **complete DevOps workflow** for a MERN stack application — from code commit to live deployment — using **GitHub Actions as the CI/CD engine** and **AWS EC2 as the deployment target**.

The pipeline automatically:
1. Triggers on every push to `main`
2. Builds **separate Docker images** for frontend (React/Nginx) and backend (Node/Express)
3. Pushes both images to **Docker Hub** with the Git SHA as the tag
4. **SSH-deploys** onto an AWS EC2 instance — pulls new images and restarts containers via Docker Compose
5. Cleans up old images to conserve disk space

---

## 🏗 Architecture

```
Developer
    │
    │  git push main
    ▼
┌──────────────────────────────────────────────────────────────┐
│                    GitHub Actions                             │
│                                                              │
│  ┌─────────────┐   ┌──────────────────┐   ┌──────────────┐  │
│  │  Checkout   │──►│  Build & Push    │──►│  SSH Deploy  │  │
│  │  Code       │   │  Docker Images   │   │  to EC2      │  │
│  └─────────────┘   │                  │   └──────┬───────┘  │
│                    │  frontend:sha    │          │          │
│                    │  backend:sha     │          │          │
│                    └──────────────────┘          │          │
└──────────────────────────────────────────────────┼──────────┘
                                                   │
                       ┌───────────────────────────┘
                       ▼
             ┌─────────────────────┐
             │     Docker Hub      │
             │  sandeeptiwari0206/ │
             │  mern-frontend:sha  │
             │  mern-backend:sha   │
             └──────────┬──────────┘
                        │  docker pull
                        ▼
┌──────────────────────────────────────────────────────────────┐
│                   AWS EC2 Instance                            │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                Docker Compose                        │   │
│   │                                                     │   │
│   │  ┌──────────────────┐    ┌─────────────────────┐   │   │
│   │  │   Frontend        │    │      Backend         │   │   │
│   │  │   React + Nginx   │◄──►│   Node.js + Express  │   │   │
│   │  │   Port: 80        │    │   Port: 5000         │   │   │
│   │  └──────────────────┘    └──────────┬───────────┘   │   │
│   │                                      │               │   │
│   └──────────────────────────────────────┼───────────────┘   │
│                                          │                   │
│                              ┌───────────▼──────────┐        │
│                              │  MongoDB Atlas        │        │
│                              │  (Cloud Database)     │        │
│                              └──────────────────────┘        │
└──────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
mernapp-githubaction-ec2/
│
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions CI/CD pipeline
│
├── frontend/                   # React application
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── App.js
│   ├── public/
│   ├── Dockerfile              # Multi-stage: build → Nginx serve
│   ├── nginx.conf              # Nginx config with API proxy
│   └── package.json
│
├── backend/                    # Node.js + Express REST API
│   ├── src/
│   │   ├── routes/
│   │   ├── models/             # Mongoose schemas
│   │   ├── controllers/
│   │   └── server.js
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml          # Production multi-service orchestration
└── .gitignore
```

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React.js, Nginx |
| **Backend** | Node.js, Express.js |
| **Database** | MongoDB Atlas (cloud) |
| **Containerisation** | Docker, Docker Compose |
| **CI/CD** | GitHub Actions |
| **Cloud** | AWS EC2 (Ubuntu) |
| **Registry** | Docker Hub |
| **Deployment** | SSH remote execution |

---

## 🔄 CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yml`) runs on every push to `main`:

```
git push main
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│  Job 1: build-and-push                                       │
│  ─────────────────────────────────────────────────────────  │
│  • actions/checkout@v3                                      │
│  • docker/login-action  → login to Docker Hub               │
│  • docker build frontend  → mern-frontend:${{ github.sha }} │
│  • docker build backend   → mern-backend:${{ github.sha }}  │
│  • docker push frontend                                     │
│  • docker push backend                                      │
└─────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│  Job 2: deploy  (needs: build-and-push)                      │
│  ─────────────────────────────────────────────────────────  │
│  • appleboy/ssh-action → SSH into EC2                       │
│  • export IMAGE_TAG=${{ github.sha }}                       │
│  • docker pull mern-frontend:$IMAGE_TAG                     │
│  • docker pull mern-backend:$IMAGE_TAG                      │
│  • docker compose down                                      │
│  • docker compose up -d                                     │
│  • docker image prune -af   (cleanup)                       │
└─────────────────────────────────────────────────────────────┘
```

### Sample Workflow YAML Structure

```yaml
name: MERN App — Build & Deploy to EC2

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build & Push Images
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/mern-backend:${{ github.sha }} ./backend
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/mern-frontend:${{ github.sha }} ./frontend
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/mern-backend:${{ github.sha }}
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/mern-frontend:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: SSH Deploy to EC2
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            export IMAGE_TAG=${{ github.sha }}
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/mern-backend:$IMAGE_TAG
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/mern-frontend:$IMAGE_TAG
            docker compose down
            docker compose up -d
            docker image prune -af
```

---

## 🚀 Getting Started

### Prerequisites

- AWS EC2 instance (Ubuntu 22.04 recommended)
- Docker & Docker Compose installed on EC2
- Docker Hub account
- MongoDB Atlas cluster URI
- GitHub repository with Actions enabled

### 1. Clone the Repository

```bash
git clone https://github.com/sandeeptiwari0206/mernapp-githubaction-ec2.git
cd mernapp-githubaction-ec2
```

### 2. Set Up EC2 Instance

```bash
# On your EC2 instance
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 3. Configure GitHub Secrets

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**

---

## 🔐 GitHub Secrets Setup

| Secret Name | Description |
|-------------|-------------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token (not password) |
| `EC2_HOST` | Public IP or DNS of your EC2 instance |
| `EC2_USER` | SSH username (usually `ubuntu`) |
| `EC2_SSH_KEY` | Contents of your EC2 `.pem` private key |
| `MONGO_URI` | MongoDB Atlas connection string |

> 💡 Generate a Docker Hub access token at: **hub.docker.com → Account Settings → Security → New Access Token**

---

## 💻 Local Development

### Run with Docker Compose

```bash
# Create a .env file
echo "MONGO_URI=your_mongodb_atlas_uri" > .env
echo "IMAGE_TAG=local" >> .env

# Start all services
docker compose up --build

# Frontend: http://localhost:80
# Backend:  http://localhost:5000
```

### Run Without Docker

```bash
# Backend
cd backend
npm install
MONGO_URI=your_uri npm run dev

# Frontend (new terminal)
cd frontend
npm install
npm start
```

---

## 👨‍💻 Author

<div align="center">

**Sandeep Tiwari** — Cloud Engineer & DevOps Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sandeep-tiwari-616a33116/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/sandeeptiwari0206)
[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-3b82f6?style=flat-square)](https://your-portfolio-url.com)

📍 Jaipur, Rajasthan, India

</div>

---

<div align="center">

⭐ **If this project helped you, give it a star!**

</div>
