# https://hub.docker.com/_/alpine
# 最新的linux镜像，只有2.68M，bash 都需要单独安装
FROM alpine:3.16
RUN apk --update add bash
RUN mkdir -p /data/code
COPY ./greeter_* /data/code/
WORKDIR /data/code/
EXPOSE 50051

CMD cd /data/code/ && ./greeter_server
