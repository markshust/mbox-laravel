FROM php:8.4-fpm-alpine

RUN adduser -u 1000 -G www-data -D appuser && \
    chown -R appuser:www-data /var/www/html

RUN apk update && apk add --no-cache \
    autoconf \
    build-base \
    bzip2-dev \
    curl \
    git \
    freetype-dev \
    icu-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxslt-dev \
    libzip-dev \
    linux-headers \
    mysql-client \
    mysql-dev \
    oniguruma-dev \
    unzip \
    zip \
    && rm -rf /var/cache/apk/*

RUN pecl channel-update pecl.php.net \
    && pecl install redis-6.1.0 \
    && pecl install xdebug-3.4.1 \
    && pecl clear-cache

RUN  docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-install \
        bcmath \
        bz2 \
        exif \
        gd \
        intl \
        mysqli \
        mbstring \
        opcache \
        pcntl \
        pdo \
        pdo_mysql \
        xsl \
        zip \
    && docker-php-ext-enable \
        redis \
        xdebug

COPY --from=composer:2.8 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

USER appuser
