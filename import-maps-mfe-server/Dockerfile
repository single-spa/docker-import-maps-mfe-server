FROM nginx

ARG importMapFilename=app.importmap

COPY app.importmap /usr/share/nginx/html/${importMapFilename}
COPY nginx.conf /etc/nginx/nginx.conf