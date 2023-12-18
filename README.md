# Dokku LimeSurvey
Por enquanto só temos o Dockerfile.

## Para testar o Dockerfile
- `DOCKER_BUILDKIT=1 docker build -t docker-limesurvey .`
- `docker run -p 80:80 docker-limesurvey`
