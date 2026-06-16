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
❯❯ docker image ls                                                              cloud-services-engineer-docker-project-sem2/backend on  main [!?⇡] via 🐹 fsh
                                                                                                                                           i Info →   U  In Use
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
dockerproject-backend:latest    f9eb3edf8589       17.4MB             0B    U
dockerproject-frontend:latest   d26a80ce6f1d       14.5MB             0B    U
```

## Configuration Frontend
- BASE_URL - базовый URL для Vue Router, по умолчанию /  
- VUE_APP_API_URL - префикс адреса API, по умолчанию /api/  
- NODE_ENV - режим сборки для Node/npm, по умолчанию development

## Configuration Backend

## Links
- [frontend](https://hub.docker.com/repository/docker/tihdizer/docker-project-frontend)
- [backend](https://hub.docker.com/repository/docker/tihdizer/docker-project-backend)
