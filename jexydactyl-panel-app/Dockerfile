FROM alpine:latest AS download
ARG jexactyl_version=v3.7.4
RUN apk add --no-cache curl tar
RUN mkdir -p /jexactyl
RUN curl -Lo panel.tar.gz https://github.com/Jexactyl/Jexactyl/releases/download/${jexactyl_version}/panel.tar.gz
RUN tar -xzvf panel.tar.gz -C /jexactyl

FROM php:8.2-fpm-alpine AS base
RUN apk add --no-cache freetype-dev libjpeg-turbo-dev libpng-dev libzip-dev git unzip \
 && docker-php-ext-configure gd --with-freetype --with-jpeg \
 && docker-php-ext-install -j$(nproc) gd pdo_mysql bcmath zip
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY php-extra.ini $PHP_INI_DIR/conf.d/extra.ini
COPY php-fpm-extra.conf /usr/local/etc/php-fpm.d/extra.conf

FROM base AS builder
WORKDIR /var/www/html
COPY --from=download /jexactyl/ .
RUN apk add --no-cache curl && curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer install --no-dev --optimize-autoloader

FROM base AS app
WORKDIR /var/www/html
COPY --from=builder /var/www/html /var/www/jex
WORKDIR /var/www/jex
COPY --chown=www-data:www-data .env.example .env
COPY cronfile /root/cronfile
RUN crontab /root/cronfile && chmod -R 755 storage bootstrap/cache
EXPOSE 80
ENTRYPOINT ["sh", "-c", "php artisan key:generate --force && php artisan migrate --force && php artisan db:seed --force && crond && chown -R www-data:www-data * && php-fpm"]
