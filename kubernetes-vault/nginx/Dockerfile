FROM nginx:1.17

MAINTAINER Vladimir Selyutin <v.selyutin@revo.ru>

RUN echo "daemon off;\n\
events {\n\
    worker_connections 1024;\n\
}\n\
http {\n\
  include /etc/nginx/mime.types;\n\
  server {\n\
    listen 443 ssl;\n\
    server_name  nginx.example.ru;\n\
    access_log off;\n\
    error_log /var/log/nginx/ngx_error.log error;\n\
    ssl_certificate      /vault/secrets/server.cert;\n\
    ssl_certificate_key  /vault/secrets/server.key;\n\
    location /basic_status {\n\
      stub_status;\n\
    }\n\
  }\n\
}\n"\
| tee /etc/nginx/nginx.conf

CMD ["nginx"]
