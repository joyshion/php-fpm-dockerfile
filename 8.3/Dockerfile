FROM alpine:3.18
LABEL maintainer="Shion <shion.chow@gmail.com>"

ENV PHP_VERSION 8.3.17

ENV BUILD_TOOLS \
    pkgconf \
    curl \
    autoconf \
    make \
    gcc \
    libc-dev

ENV BUILD_DEPS \
    libxml2-dev \
    curl-dev \
    sqlite-dev \
    gd-dev \
    gettext-dev \
    gmp-dev \
    oniguruma-dev \
    postgresql-dev \
    libedit-dev \
    libzip-dev \
    libxpm-dev

ENV RUNTIME_DEPS \
    libpq \
    gmp \
    libintl \
    libxml2 \
    sqlite-libs \
    libcurl \
    libpng \
    libwebp \
    libjpeg-turbo \
    libxpm \
    freetype \
    oniguruma \
    libedit \
    libedit \
    libzip \
    icu-libs

ENV PHP_INI_DIR /usr/local/etc/php

# 82 is the standard uid/gid for "www-data" in Alpine
RUN adduser -u 82 -D -H -S -G www-data www-data

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
    && curl -o php.tar.gz -L https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz \
    && tar -zxf php.tar.gz \
    && cd php-${PHP_VERSION} \
    && ./configure \
        --with-config-file-path=$PHP_INI_DIR \
        --with-config-file-scan-dir=$PHP_INI_DIR/conf.d \
        --enable-fpm \
        --with-fpm-user=www-data \
        --with-fpm-group=www-data \
        --disable-cgi \
        --disable-short-tags \
        --with-openssl \
        --with-pcre-jit \
        --with-zlib \
        --enable-bcmath \
        --with-curl \
        --enable-exif \
        --enable-gd \
        --with-webp \
        --with-jpeg \
        --with-xpm \
        --with-freetype \
        --with-gettext \
        --with-gmp \
        --enable-mbstring \
        --with-pdo-mysql \
        --with-pdo-pgsql \
        --with-libedit \
        --with-zip \
        --enable-mysqlnd \
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
    # redis
    && cd /usr/src \
    && curl -o redis.tar.gz -L https://pecl.php.net/get/redis \
    && tar -zxvf redis.tar.gz \
    && cd redis-* \
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