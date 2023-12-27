FROM php:8.2-fpm

ENV DEBIAN_FRONTEND noninteractive
ENV COMPOSER_ALLOW_SUPERUSER 1

RUN apt-get update && apt-get install -y \
    postgresql-client \
    zip \
    unzip \
    p7zip-full \
    git \
    wget \
    libwebp-dev \
    libpng-dev \
    libjpeg-dev \
    nano

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN install-php-extensions gd xdebug zip pdo_mysql intl opcache apcu

WORKDIR /var/www/html

COPY ./docker/php/php.ini-development /usr/local/etc/php/php.ini
COPY ./docker/php/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./drupal-dev/composer.json /var/www/html/composer.json
COPY ./drupal-dev/composer.lock /var/www/html/composer.lock
COPY ./drupal-dev/web/themes/custom/ /var/www/html/web/themes/custom/
COPY ./drupal-dev/web/modules/custom/ /var/www/html/web/modules/custom/
COPY ./drupal-dev/config /var/www/html/config

RUN composer install --no-interaction --no-progress --no-suggest --optimize-autoloader
RUN composer require drush/drush --no-interaction
RUN echo "alias drush='/var/www/html/vendor/bin/drush'" >> ~/.bashrc

RUN  head -n -4 /var/www/html/web/sites/default/default.settings.php > /var/www/html/web/sites/default/settings.php

RUN echo 'if (file_exists($app_root . "/" . $site_path . "/settings.local.php")) {' >> /var/www/html/web/sites/default/settings.php
RUN echo '   include $app_root . "/" . $site_path . "/settings.local.php";' >> /var/www/html/web/sites/default/settings.php
RUN echo '}' >> /var/www/html/web/sites/default/settings.php

RUN chown -R www-data:www-data /var/www/html
RUN chmod -R 770 /var/www/html

RUN usermod -a -G www-data root
