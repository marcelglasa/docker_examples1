# Docker examples
Docker configuration files examples

## RabbitMQ on Windows Nano server

[Dockerfile](RabbitMQ/Docker_Windows_NANO/Dockerfile) based on image mcr.microsoft.com/powershell:nanoserver-1809, RABBITMQ_VERSION=3.8.2, OTP_VERSION=22.2, [rabbitmq.conf](RabbitMQ/Docker_Windows_NANO/rabbitmq.conf), [enabled_plugins](RabbitMQ/Docker_Windows_NANO/enabled_plugins)

build:
```
docker build --build-arg RABBITMQ_VERSION=3.8.2 -f Dockerfile -t IMAGE_NAME .
```
run:
```
docker run -d --name CONTAINER_NAME IMAGE_NAME
```
view status:
```
docker container exec CONTAINER_NAME cmd /c rabbitmqctl status
```
view enabled plugins:
```
docker exec -it CONTAINER_NAME cmd /c rabbitmq-plugins list
```
