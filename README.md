# Local Development Services – Clean & Professional Setup

This setup is designed to help you **run all common backend services in one place with a single Docker command**, while keeping things clean, scalable, and easy to manage.

The goal is:

* One folder
* One `docker-compose up`
* Clear responsibility per service
* Easy to extend later (Kafka, ELK, etc.)

---

## 1. Folder Structure (Professional & Scalable)

```
services/
├── docker-compose.yml
├── .env
├── README.md
│
├── mysql/
│   └── init/
│       └── init.sql
│
├── redis/
│   └── redis.conf
│
├── minio/
│   └── data/
│
├── keycloak/
│   └── realm-export.json
│
└── nginx/
    └── nginx.conf
```

Why this structure:

* **Each service owns its own config**
* Easy to debug or replace one service
* Looks professional in real projects

---

## 2. Environment Variables (`.env`)

```
# MySQL
MYSQL_ROOT_PASSWORD=root123
MYSQL_DATABASE=app_db
MYSQL_USER=app_user
MYSQL_PASSWORD=app123

# MinIO
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123

# Keycloak
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin123
```

Rule: **Never hardcode secrets in docker-compose**.

---

## 3. Docker Compose (Main Control Center)

```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: always
    env_file: .env
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/init:/docker-entrypoint-initdb.d

  redis:
    image: redis:7
    container_name: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  minio:
    image: minio/minio
    container_name: minio
    restart: always
    env_file: .env
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data

  keycloak:
    image: quay.io/keycloak/keycloak:24.0
    container_name: keycloak
    command: start-dev
    env_file: .env
    ports:
      - "8080:8080"
    depends_on:
      - mysql

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - keycloak

volumes:
  mysql_data:
  redis_data:
  minio_data:
```

---

## 4. How to Run Everything (One Command)

```
docker compose up -d
```

To stop:

```
docker compose down
```

To reset all data:

```
docker compose down -v
```

---

## 5. Service Access URLs (Local)

| Service  | URL                                            |
| -------- | ---------------------------------------------- |
| MySQL    | localhost:3306                                 |
| Redis    | localhost:6379                                 |
| MinIO    | [http://localhost:9001](http://localhost:9001) |
| Keycloak | [http://localhost:8080](http://localhost:8080) |
| Nginx    | [http://localhost](http://localhost)           |

---

## 6. Mental Model (Very Important)

Think like this:

* `docker-compose.yml` → **Orchestrator**
* `services/*` folders → **Service ownership**
* `.env` → **Configuration contract**
* volumes → **Persistent data**

This mindset is how real DevOps & backend teams work.

---

## 7. Next Learning Steps (Recommended)

Once you’re comfortable:

* Add **Kafka**
* Add **Prometheus + Grafana**
* Connect your **Spring Boot / FastAPI** app
* Use **profiles** for dev / prod

---

## 8. README.md (Senior Engineer Style)

```md
# Local Development Services

A unified local development environment that runs all required backend services using Docker Compose.

This repository is intended for **learning, local development, and system design practice**, following clean and professional engineering standards.

---

## Overview

This setup provides commonly used backend infrastructure services in one place, allowing developers to:

- Start all services with a single command
- Maintain consistent environments across machines
- Easily add or remove services
- Learn how real-world backend systems are structured

The environment is designed to be **simple, explicit, and production-minded**.

---

## Included Services

| Service   | Purpose |
|---------|--------|
| MySQL    | Primary relational database |
| Redis    | Caching, session storage |
| MinIO    | S3-compatible object storage |
| Keycloak | Authentication & authorization (OAuth2/OIDC) |
| Nginx    | Reverse proxy / entry point |

---

## Folder Structure

```

services/
├── docker-compose.yml     # Main orchestration file
├── .env                  # Environment variables (DO NOT commit secrets)
├── README.md             # Documentation
│
├── mysql/                # MySQL-specific configuration
├── redis/                # Redis configuration
├── minio/                # Object storage data
├── keycloak/             # Auth realm configuration
└── nginx/                # Reverse proxy configuration

````

Each service owns its configuration to ensure:
- Clear responsibility
- Easier debugging
- Safe scalability

---

## Prerequisites

Make sure the following are installed:

- Docker (v20+)
- Docker Compose (v2+)

Verify installation:

```bash
docker --version
docker compose version
````

---

## Getting Started

1. Clone the repository
2. Create environment variables

```bash
cp .env.example .env
```

3. Start all services

```bash
docker compose up -d
```

4. Verify running containers

```bash
docker compose ps
```

---

## Service Access

| Service  | URL                                            |
| -------- | ---------------------------------------------- |
| MySQL    | localhost:3306                                 |
| Redis    | localhost:6379                                 |
| MinIO    | [http://localhost:9001](http://localhost:9001) |
| Keycloak | [http://localhost:8080](http://localhost:8080) |
| Nginx    | [http://localhost](http://localhost)           |

---

## Common Commands

Start services:

```bash
docker compose up -d
```

Stop services:

```bash
docker compose down
```

Stop and remove all data:

```bash
docker compose down -v
```

View logs:

```bash
docker compose logs -f
```

---

## Design Principles

* **Single Responsibility**: One container, one purpose
* **Explicit Configuration**: No hidden magic
* **Environment Parity**: Local mirrors real systems
* **Easy Reset**: Stateless services, persistent volumes

---

## Extending the Setup

Common next additions:

* Kafka / Zookeeper (event-driven systems)
* Prometheus + Grafana (monitoring)
* Application services (Spring Boot, FastAPI)
* Docker Compose profiles (dev / prod)

---

## Notes

* This setup is for **local development only**
* Do not reuse credentials in production
* Always review exposed ports before deploying elsewhere

---

## Author

Prepared with a focus on clean architecture, operational clarity, and long-term maintainability.

```

---

You now have a **production-quality README** that would be acceptable in real teams and interviews.

Next steps if you want to level up further:
- Add diagrams (architecture & data flow)
- Add Makefile for commands
- Add CI checks (lint, compose validation)

Just tell me where you want to go next.

```
