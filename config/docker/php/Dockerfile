FROM php:7.4-fpm-alpine

RUN apk add --no-cache bash vim git \
    openssl \
    libcurl curl curl-dev \
    autoconf gcc g++ make \
    zlib-dev \
    libmemcached-dev \
    openssh

RUN docker-php-ext-install sockets
RUN docker-php-ext-install json
RUN docker-php-ext-install ctype
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install curl
RUN docker-php-ext-install bcmath

### ZIP - START
RUN apk add --no-cache zip libzip-dev
RUN docker-php-ext-configure zip
RUN docker-php-ext-install zip
### ZIP - END

### GD - START
RUN apk add --no-cache freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev
RUN docker-php-ext-configure gd
RUN docker-php-ext-install gd
### GD - END

RUN docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install opcache

COPY --chown=www-data:www-data config/docker/ /tmp/

RUN mv /tmp/php/php.ini /usr/local/etc/php/ && \
    mv /tmp/php/www.conf /usr/local/etc/php-fpm.d/ && \
    rm -rf /tmp/* /var/www/html/ && \
    chown -R www-data:www-data /usr/local/etc/php

RUN \
    pecl install apcu && \
    pecl install redis && \
    pecl install memcached && \
    docker-php-ext-enable apcu && \
    docker-php-ext-enable redis && \
    docker-php-ext-enable memcached

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN pecl install xdebug && docker-php-ext-enable xdebug

COPY --chown=www-data:www-data . /data
USER www-data
WORKDIR /data

HEALTHCHECK --interval=10s --timeout=3s \
    CMD \
    SCRIPT_NAME=/ping \
    SCRIPT_FILENAME=/ping \
    REQUEST_METHOD=GET \
    cgi-fcgi -bind -connect 127.0.0.1:9000 || exit 1

EXPOSE 9000

STOPSIGNAL SIGQUIT

CMD ["php-fpm"]
