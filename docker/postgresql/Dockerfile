FROM postgres:13.5

ADD ./docker/postgresql/extensions/extension.sql /docker-entrypoint-initdb.d/extension.sql

WORKDIR /var/lib/postgresql/dump/