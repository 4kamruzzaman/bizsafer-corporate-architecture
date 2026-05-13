# Reference Architecture: Laravel 13 API Core
FROM php:8.4-fpm-alpine

# ---------------------------------------------------------
# System Dependencies & PHP Extensions
# ---------------------------------------------------------
RUN apk add --no-cache \
    postgresql-dev \
    libzip-dev \
    unzip \
    && docker-php-ext-install pdo pdo_pgsql zip opcache pcntl

# Install Redis Extension
RUN apk add --no-cache pcre-dev $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del $PHPIZE_DEPS

# ---------------------------------------------------------
# Application Configuration
# ---------------------------------------------------------
WORKDIR /var/www/html
COPY . .

# Install Composer Dependencies
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Enforce Strict Permissions
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

# ---------------------------------------------------------
# Production Caching Optimization
# ---------------------------------------------------------
RUN php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache

EXPOSE 9000
CMD ["php-fpm"]
