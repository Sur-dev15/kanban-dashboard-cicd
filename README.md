# Kanban Dashboard – Docker, Jenkins & AWS CI/CD

## Project Overview

This project demonstrates automated deployment of a React + TypeScript Kanban Dashboard using Docker, Jenkins, GitHub, Docker Hub, and AWS EC2.

The CI/CD pipeline automatically builds the application, creates a Docker image, pushes the image to Docker Hub, deploys it to AWS EC2, and validates application health.

## Technology Stack

- React
- TypeScript
- Vite
- Docker
- Jenkins
- GitHub
- Docker Hub
- AWS EC2
- Nginx

## Application Details

- Local development port: `5173`
- Docker container port: `8080`
- Production HTTP port: `80`
- Build command: `npm run build`
- Build output directory: `dist`
- Environment variables: None required

## Run Locally

Prerequisites: Node.js and npm.

```bash
npm install
npm run dev
```

Open `http://localhost:5173`.

Build the application:

```bash
npm run build
```

## Docker Configuration

The Dockerfile uses a multi-stage build:

1. Node.js Alpine builds the React application.
2. Nginx Unprivileged serves the production build.
3. The container runs as non-root user UID 101.
4. Docker HEALTHCHECK monitors application health.

Build the image:

```bash
docker build -t kanban-dashboard .
```

Run the container:

```bash
docker run -
