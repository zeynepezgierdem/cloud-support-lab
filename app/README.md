# Hello API

A minimal Flask service with a single `/health` endpoint, built to practice
containerizing and running a real application with Docker.

## Run locally with Docker

```bash
docker build -t hello-api .
docker run -d --name hello-api -p 5000:5000 hello-api
curl http://localhost:5000/health
```

Expected response:

```json
{"status": "ok"}
```

## Stop and remove

```bash
docker stop hello-api
docker rm hello-api
```
