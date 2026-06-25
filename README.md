# 🌐 Nginx Reverse Proxy / API Gateway

![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

This repository acts as the **Central Hub (API Gateway)** for my microservices infrastructure. It utilizes Nginx running on Docker to intercept web traffic (Port 80) and route it internally to different applications, which are isolated within their own containers and subnets.

## 🏗️ System Architecture

The centralized proxy allows multiple applications to coexist on the same physical host (Ubuntu Server) without port collisions, ensuring high availability and independent deployments.

- **Frontend / Hub UI:** Serves a static interface (`index.html`) acting as the main dashboard.
- **Routing:** Intercepts incoming requests and performs a reverse proxy to external Docker networks (`global-router`).

## 🔗 Connected Microservices Ecosystem

This proxy actively routes traffic to the following independent services:

1. **[IT Inventory System](PON_AQUI_EL_LINK_A_TU_OTRO_REPO):** Spring Boot (Java 17) and MySQL backend for IT hardware management. Routed under `/inventario/`.
