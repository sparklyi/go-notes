## jaeger笔记



### docker安装

```sh
docker run \
  --rm \
  --name jaeger \
  -d\
  -p6831:6831/udp \
  -p16686:16686 \
  -p14268:14268 \
  jaegertracing/all-in-one:latest

```

