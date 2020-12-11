FROM alpine:3.12
MAINTAINER Shion <shion.chow@gmail.com>

ENV PHP_VERSION 8.0.0
ENV PHP_MEMCACHED_VERSION 3.1.5
ENV PHP_REDIS_VERSION 5.3.2

ENV BUILD_TOOLS \
        autoconf \
        make \
        gcc \
        g++ \
        file \
        re2c \
        curl

ENV BUILD_DEPS \
        sqlite-dev \
        curl-dev \
        pcre2-dev \
        bzip2-dev \
        libxml2-dev \
        gd-dev \
        oniguruma-dev \
        libzip-dev \
        libmemcached-dev

ENV PHP_INI_DIR /usr/local/etc/php

# 82 is the standard uid/gid for "www-data" in Alpine
RUN addgroup -g 82 -S www-data && adduser -u 82 -D -S -G www-data www-data

# https://github.com/docker-library/php/issues/272
RUN export CFLAGS="-fstack-protector-strong -fpic -fpie -O2" \
  && export CPPFLAGS=${CFLAGS} \
  && export LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie" \
  && export MAKEFLAGS="-j $(getconf _NPROCESSORS_ONLN)"

RUN apk add --no-cache --virtual .build-tools $BUILD_TOOLS \
    && apk add --no-cache --virtual .build-deps $BUILD_DEPS \
    && apk add --no-cache --virtual .runtime-deps $RUNTIME_DEPS \
    && mkdir -p $PHP_INI_DIR/conf.d \
    && mkdir -p /usr/src \
    && cd /usr/src \
    && curl -o php.tar.gz -L https://php.net/get/php-${PHP_VERSION}.tar.gz/from/this/mirror \
    && tar -zxf php.tar.gz \
    && cd php-${PHP_VERSION} \
    && ./configure \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --enable-fpm \
        --with-fpm-user=www-data \
        --with-fpm-group=www-data \
        --disable-short-tags \
        --with-openssl \
        --with-external-pcre \
        --with-pcre-jit \
        --with-zlib \
        --enable-bcmath \
        --with-bz2 \
        --enable-calendar \
        --with-curl \
        --disable-dom \
        --enable-exif \
        --with-openssl-dir \
        --enable-gd \
        --with-external-gd \
        --with-webp \
        --with-xpm \
        --with-jpeg \
        --with-freetype \
        --with-libedit \
        --enable-mbstring \
        --with-pdo-mysql \
        --with-zlib-dir \
        --with-zip \
        --enable-mysqlnd \
        --disable-fileinfo \
    && make \
    && make install \
    # config
    && cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf \
    && cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/;error_log = log\/php-fpm.log/error_log = \/proc\/self\/fd\/2/g' /usr/local/etc/php-fpm.conf \
    && sed -i 's/;daemonize = yes/daemonize = no/g' /usr/local/etc/php-fpm.conf \
    && sed -i 's!=NONE/!=!g' /usr/local/etc/php-fpm.conf \
    && sed -i 's/listen = 127.0.0.1:9000/listen = [::]:9000/g' /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/;access.log = log\/\$pool.access.log/access.log = \/proc\/self\/fd\/2/g' /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/;catch_workers_output = yes/catch_workers_output = yes/g' /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/;clear_env = no/clear_env = yes/g' /usr/local/etc/php-fpm.d/www.conf \
    && echo "zend_extension=opcache.so" >> $PHP_INI_DIR/conf.d/opcache.ini \
    # memcached
    && cd /usr/src \
    && curl -o memcached.tar.gz -L https://github.com/php-memcached-dev/php-memcached/archive/v${PHP_MEMCACHED_VERSION}.tar.gz \
    && tar -zxvf memcached.tar.gz \
    && cd php-memcached-${PHP_MEMCACHED_VERSION} \
    && phpize \
    && ./configure --enable-memcached-json --disable-memcached-sasl \
    && make \
    && make install \
    && echo "extension=memcached.so" >> $PHP_INI_DIR/conf.d/memcached.ini \
    # redis
    && cd /usr/src \
    && curl -o redis.tar.gz -L https://github.com/phpredis/phpredis/archive/${PHP_REDIS_VERSION}.tar.gz \
    && tar -zxvf redis.tar.gz \
    && cd phpredis-${PHP_REDIS_VERSION} \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=redis.so" >> $PHP_INI_DIR/conf.d/redis.ini \
    # clear
    && cd ~ && rm -rf /usr/src && apk del .build-tools .build-deps

VOLUME ["/app"]
WORKDIR /app

EXPOSE 9000
CMD ["php-fpm"]
