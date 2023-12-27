FROM nginx:latest

RUN apt update -y
RUN apt install openssl ufw acl procps gettext-base -y

WORKDIR /etc/ssl/private
RUN openssl genrsa -des3 -passout pass:x -out server.pass.key 2048 && \
    openssl rsa -passin pass:x -in server.pass.key -out server.key && \
    rm server.pass.key && \
    openssl req -new -key server.key -out server.csr \
        -subj "/C=HU/ST=Fejer/L=Budapest/O=Lechner/OU=IT Department/CN=dev.tozsde.local" && \
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
RUN openssl dhparam -out /etc/nginx/dhparam.pem 4096
RUN mkdir /etc/nginx/snippets
RUN echo "ssl_certificate /etc/ssl/private/server.crt;" > /etc/nginx/snippets/self-signed.conf
RUN echo "ssl_certificate_key /etc/ssl/private/server.key;" >> /etc/nginx/snippets/self-signed.conf

RUN echo "ssl_prefer_server_ciphers on;" > /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_protocols TLSv1.3;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_dhparam /etc/nginx/dhparam.pem;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_ciphers EECDH+AESGCM:EDH+AESGCM;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_ecdh_curve secp384r1;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_session_timeout  10m;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_session_cache shared:SSL:10m;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_session_tickets off;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_stapling on;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "ssl_stapling_verify on;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "resolver 8.8.8.8 8.8.4.4 valid=300s;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "resolver_timeout 5s;" >> /etc/nginx/snippets/ssl-params.conf
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
RUN echo "add_header X-Frame-Options DENY;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "add_header X-Content-Type-Options nosniff;" >> /etc/nginx/snippets/ssl-params.conf
RUN echo "add_header X-XSS-Protection "1; mode=block";" >> /etc/nginx/snippets/ssl-params.conf
RUN ufw allow 'WWW Full'

COPY ./docker/nginx/default.conf /etc/nginx/conf.d/default.conf

WORKDIR /var/www/html
