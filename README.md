# 🌐 Nginx Reverse Proxy / API Gateway

![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

This repository acts as the **Central Hub (API Gateway)** for my microservices infrastructure. It utilizes Nginx running on Docker to intercept web traffic (Port 80) and route it internally to different applications, which are isolated within their own containers and subnets.

## 🏗️ System Architecture

The centralized proxy allows multiple applications to coexist on the same physical host (Ubuntu Server) without port collisions, ensuring high availability and independent deployments.

- **Frontend / Hub UI:** Serves a static interface (`html/index.html`) acting as the main dashboard.
- **Routing:** Intercepts incoming requests and performs a reverse proxy to containers attached to the external Docker network `global-router`.

```text
Browser ──:80──▶ hub-proxy
                  ├─ /             → html/index.html
                  └─ /inventario/  → inventario-frontend:80   (prefix stripped)
                                       └─ /api/ → inventario-backend:8080
```

Each app is reached only through its own prefix. The hub does not route any `/api/...` path at the root, so apps never collide on API paths.

## 🔗 Connected Microservices Ecosystem

This proxy actively routes traffic to the following independent services:

1. **[IT Inventory System](https://github.com/AlexSuarez-dev/Centralized-data-structure.git):** Spring Boot (Java 17) and MySQL backend for IT hardware management. Routed under `/inventario/` (API under `/inventario/api/v1/`, Swagger at `/inventario/api/docs`).

## 🚀 Running

```bash
docker network create global-router   # once per server
docker compose up -d
docker exec hub-proxy nginx -s reload  # after editing nginx.conf or html/
```

Deploy order: when an app changes its routes, deploy the app first and the hub afterwards.

## ➕ Adding a new app

1. In the app's `docker-compose.yml`: set a `container_name`, attach the public-facing container to the external `global-router` network and do not publish ports to the host.
2. In `nginx.conf`, add a block following the inventory pattern:

   ```nginx
   location = /myapp { return 301 /myapp/; }
   location /myapp/ {
       set $upstream_myapp http://myapp-container:80;
       rewrite ^/myapp/(.*) /$1 break;
       proxy_pass $upstream_myapp;
       proxy_set_header Host $host;
       proxy_set_header X-Forwarded-Prefix /myapp;
   }
   ```

   Using a variable in `proxy_pass` (with `resolver 127.0.0.11`) lets Nginx start even if the app is down.
3. The app must use **relative URLs** (`api/...`, not `/api/...`) so every request stays under its prefix.
4. Add the link to `html/index.html` and reload Nginx.
