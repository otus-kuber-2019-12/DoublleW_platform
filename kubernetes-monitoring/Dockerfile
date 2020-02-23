FROM alpine:3.4

MAINTAINER Vladimir Selyutin <v.selyutin@revo.ru>

RUN apk add --update nginx && rm -rf /var/cache/apk/*

RUN echo -e "daemon off;\n\
pid /var/lib/nginx/nginx.pid;\n\
events {\n\
    worker_connections 1024;\n\
}\n\
http {\n\
  include /etc/nginx/mime.types;\n\
  index index.html index.htm;\n\
  server {\n\
    listen *:8088;\n\
    access_log off;\n\
    error_log /var/log/nginx/ngx_error.log error;\n\
    location /basic_status {\n\
      stub_status;\n\
    }\n\
  }\n\
}\n"\
| tee /etc/nginx/nginx.conf

EXPOSE 8088

CMD ["nginx"]
