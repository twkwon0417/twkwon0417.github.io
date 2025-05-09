FROM nginx

ARG HOST_FILES
COPY ${HOST_FILES} /usr/share/nginx/html
COPY ../config/nginx.conf /etc/nginx/nginx.conf

EXPOSE 9000
