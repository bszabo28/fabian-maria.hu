FROM node:slim

RUN mkdir -p /app
WORKDIR /app

COPY ./drupal-dev/frontend /app
RUN npm install --silent
RUN npm install --global gulp-cli --silent