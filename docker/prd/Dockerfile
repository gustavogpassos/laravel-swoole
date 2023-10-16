FROM php:8.2.4-bullseye

## Diretório da aplicação
ARG APP_DIR=/var/www/app

## Versão da Lib do Redis para PHP
ARG REDIS_LIB_VERSION=5.3.7

### apt-utils é um extensão de recursos do gerenciador de pacotes APT
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    apt-utils \
    supervisor

# dependências recomendadas de desenvolvido para ambiente linux
RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libzip-dev \
    unzip \
    libpng-dev \
    libpq-dev \
    libxml2-dev

#RUN docker-php-ext-install mysqli pdo pdo_mysql pdo_pgsql pgsql session xml
RUN docker-php-ext-install pdo pdo_mysql session xml

# habilita instalação do Redis
RUN pecl install redis-${REDIS_LIB_VERSION} \
    && docker-php-ext-enable redis

RUN docker-php-ext-install zip iconv simplexml pcntl gd fileinfo

### Instalar e Habilitar o Swoole
RUN pecl install swoole
RUN docker-php-ext-enable swoole

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY ./docker/prd/supervisord.octane.conf /etc/supervisor/conf.d/supervisord.conf

COPY ./docker/prd/extra-php.ini "$PHP_INI_DIR/99_extra.ini"
# COPY ./docker/php/extra-php-fpm.conf /etc/php8/php-fpm.d/www.conf

WORKDIR $APP_DIR
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR

COPY --chown=www-data:www-data ./ .

RUN composer install

### OCTANE
RUN composer require laravel/octane
RUN php artisan octane:install --server=swoole

### Comandos úteis para otimização da aplicação
RUN php artisan clear-compiled
RUN php artisan optimize

RUN php artisan config:clear

### NGINX
RUN apt-get install nginx -y
RUN rm -rf /etc/nginx/sites-enabled/* && rm -rf /etc/nginx/sites-available/*
COPY ./docker/prd/sites.octane.conf /etc/nginx/sites-enabled/default.conf

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN chmod 777 *

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
