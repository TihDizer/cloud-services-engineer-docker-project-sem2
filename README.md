## Prerequisites

Docker and docker compose

## Run

```bash
docker compose up -d
```

После запуска приложение доступно по адресу http://localhost/

## Optimisations

Используются multi-stage сборки и легкие базовые образы

backend:
- build: использует `golang:1.17-alpine`;
- run: `alpine:3.24.1`;
- бинарник собирается с флагами `-ldflags="-s -w"`, чтобы не переносить Go toolchain и уменьшить размер исполняемого файла.

frontend:

- build: `node:24.13.0-alpine`;
- run: `nginx:1.30-alpine3.23-slim`;
- в финальный образ копируется только собранная директория `dist`, без `node_modules` и исходников.

Размеры итоговых образов:
```bash
❯❯ docker image ls                                                                cloud-services-engineer-docker-project-sem2/backend on  main [!] via 🐹 fsh
IMAGE                                     ID             DISK USAGE   CONTENT SIZE   EXTRA
tihdizer/docker-project-backend:latest    2a50bf70535f       22.7MB             0B    U
tihdizer/docker-project-frontend:latest   0b70f8d3f344       19.8MB             0B    U
```

## Configuration Frontend
build:
- BASE_URL - базовый URL для Vue Router, по умолчанию /  
- VUE_APP_API_URL - префикс адреса API, по умолчанию /api/  
- NODE_ENV - режим сборки для Node/npm, по умолчанию development
run:
- FRONTEND_CPUS - количество CPU для контейнера frontend 
- FRONTEND_RAM - количество RAM для контейнера frontend 
- UID - id пользователя для запуска nginx (по умолванию 1001)
## Configuration Backend
run:
- BACKEND_CPUS - количество CPU для контейнера backend
- BACKEND_RAM -  количество RAM для контейнера backend
- UID - id пользователя для запуска бинарника backend (по умолванию 1001)

## Links
- [frontend](https://hub.docker.com/repository/docker/tihdizer/docker-project-frontend)
- [backend](https://hub.docker.com/repository/docker/tihdizer/docker-project-backend)

## Starting order
В docker-compose.yml для сервиса frontend задано depends_on относительно backend с условием service_healthy. Сначала поднимается и проходит проверку готовности backend (эндпоинт /health), после этого запускается контейнер frontend. 

## Scaling
Несколько экземпляров API могут подниматься как реплики сервиса backend в Docker Compose. Запросы с фронта на /api/ проксирует nginx из контейнера frontend

Пример запуска с тремя репликами backend: 
```bash
docker compose up -d --build --scale backend=3
```

## Security

Для запуска контейнеров frontend и backend используются непривилегированные пользователи frontend:nginx и backend:www-data соответственно.
