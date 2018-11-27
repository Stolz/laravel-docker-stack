FROM php:fpm-alpine

# Install Postgres PHP extension
RUN apk --no-cache add postgresql-dev && docker-php-ext-install pdo_pgsql

# Install Supervisor
RUN apk --no-cache add supervisor && mkdir /etc/supervisor.d

# Create user and group
RUN addgroup -S -g 1000 www && adduser -S -D -u 1000 -G www www

# Create workdir
RUN mkdir /www && touch /www/docker-volume-not-mounted && chown www:www /www
WORKDIR /www

# Supervisor will run PHP-FPM and Laravel queue workers
CMD ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
