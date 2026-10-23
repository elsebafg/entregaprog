# container-demo

[![CI/CD](https://github.com/elsebafg/entregaprog/actions/workflows/ci.yml/badge.svg)](https://github.com/elsebafg/entregaprog/actions/workflows/ci.yml)
[![Codacy Badge](https://app.codacy.com/project/badge/Grade/05eda3c2896b4c29ba08ced5b2399dae)](https://app.codacy.com/gh/elsebafg/entregaprog/dashboard)
[![Docker Hub](https://img.shields.io/docker/v/sebafloresg/container-demo?label=docker%20hub&sort=semver)](https://hub.docker.com/r/sebafloresg/container-demo)

App Node.js + TypeScript contenerizada con Docker, con pipeline de CI/CD en GitHub Actions
que la analiza (Codacy), la construye y publica la imagen en Docker Hub con version automatica.

## Endpoints

| Ruta      | Respuesta                                                              |
|-----------|------------------------------------------------------------------------|
| `/`       | `{ "message": "Hola desde un contenedor", "hostname": "...", "node": "..." }` |
| `/health` | `{ "status": "healthy" }`                                              |
| `/info`   | `{ "app": "container-demo", "version": "2.0", "containerized": true }` |

La `version` de `/info` sale de la variable `APP_VERSION`, que el pipeline inyecta al construir la imagen.

## Correr en local

```bash
npm install
npm run build
npm start          # http://localhost:3000
```

## Correr con Docker

```bash
docker build -t container-demo:2.0 .
docker run -d --name container-demo -p 3000:3000 container-demo:2.0
curl http://localhost:3000/info
```

Desde Docker Hub (sin clonar nada):

```bash
docker pull sebafloresg/container-demo:latest
docker run -d --name demo -p 3000:3000 sebafloresg/container-demo:latest
```

## Pipeline (`.github/workflows/ci.yml`)

| Job      | Cuando                | Que hace                                                                 |
|----------|-----------------------|--------------------------------------------------------------------------|
| `build`  | push y PR a `main`    | `npm ci`, compila con `tsc` y prueba `/health` e `/info`                 |
| `codacy` | push y PR a `main`    | Inspeccion de codigo estatico con Codacy                                 |
| `deploy` | solo push a `main`    | Arma la imagen y la publica en Docker Hub con el tag de la nueva version |

**Version automatica (bonus):** cada deploy toma el ultimo tag de git `vX.Y`, le suma 1 a `Y`
(`2.0` → `2.1` → `2.2` …), publica la imagen como `sebafloresg/container-demo:X.Y` y `:latest`,
y registra el nuevo tag `vX.Y` en el repo. El primer deploy publica `2.0`.

### Secrets necesarios (Settings → Secrets and variables → Actions)

| Secret                  | Para que                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| `DOCKERHUB_TOKEN`       | Access token de Docker Hub con permisos *Read & Write*                   |
| `CODACY_PROJECT_TOKEN`  | Opcional. Project API token de Codacy para subir los resultados del CI   |
