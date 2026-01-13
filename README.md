# AWS QuizApp DevSecOps project

This guide walks through building, containerizing, and deploying a Flask-based AWS Quiz App using Docker, GitHub Actions CI/CD (with pytest and Trivy), Raspberry Pi hosting, Caddy reverse proxy, and a custom domain. Start from scratch on a fresh Ubuntu Server

## Project Background

During the **initial phase of my training within the ITS DevSecOps program (EQF Level 5)** provided by the **Cisco Academy A.Olivetti**, I developed a web application exclusively for **personal use**, with the goal of preparing for the **AWS Cloud Practitioner** certification.  
The application was designed as a self-study tool and is based on approximately **400 exam-style questions**, featuring a basic Python backend.

**At a later stage of the program**, after completing the module on **Containers and Docker**, I expanded the project by deploying the application alongside additional containerized services (such as WireGuard , Nextcloud and AdGuard) on an **Ubuntu Server VirtualBox's VM**.  
This phase allowed me to gain hands-on experience with Docker, service isolation, networking, and basic infrastructure management.

**In a next phase**, the entire infrastructure was migrated to a **Raspberry Pi 5**, where the architecture was significantly improved and restructured.  
The system now follows a **Docker-based microservices approach**, using **Caddy as a reverse proxy**, **Cloudflare Dynamic DNS**, and a **custom domain("enrisox-devops.it")**.  
This evolution transformed a personal study application into a publicly accessible project aligned with modern **DevOps and cloud-native practices**, including automated CI/CD workflows and security-oriented design principles.

## Deployment Environment

- GitHub account with a dedicated repository.
- Docker Hub account for container image publishing (Docker Hub or AWS ECR).
- Raspberry Pi running **Ubuntu Server**.
- Docker and Docker Compose installed.
- **Caddy** reverse proxy configured on  a dedicated and isolated Docker network.
- Custom domain managed via **Cloudflare**.
- SSH access to the Raspberry Pi with a dedicated `deploy` user for improved security.

## Docker vs Podman
Docker was selected over Podman despite its rootless capabilities.
While Podman would provide a stronger security model, Docker was chosen to ensure compatibility with GitHub Actions, multi-architecture builds, and ecosystem maturity.
The associated risk is mitigated by restricting access to the Docker daemon and limiting the runner to a trusted single-host environment.

Possible future improvements: migration to Podman rootless or Docker Swarm/Kubernetes for containers orchestration.
