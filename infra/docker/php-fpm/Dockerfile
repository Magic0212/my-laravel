# ---- stage 1: composer ----
    FROM composer:2 AS vendor
    WORKDIR /app
    COPY app/composer.json app/composer.lock ./
    RUN composer install --no-dev --prefer-dist --no-scripts --no-interaction
    COPY app/ ./
    RUN composer dump-autoload --optimize --no-dev
    
    # ---- stage 2: php-fpm ----
    FROM php:8.2-fpm-alpine
    RUN apk add --no-cache git zip unzip oniguruma-dev icu-dev libzip-dev \
     && docker-php-ext-configure intl \
     && docker-php-ext-install pdo pdo_mysql intl zip \
     && { echo "opcache.enable=1"; echo "opcache.jit=disable"; } > /usr/local/etc/php/conf.d/opcache.ini
    WORKDIR /var/www/html
    COPY --from=vendor /app /var/www/html
    RUN chown -R www-data:www-data storage bootstrap/cache
    USER www-data
    EXPOSE 9000
    CMD ["php-fpm", "-F"]
    