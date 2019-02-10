FROM php:fpm-alpine

ENV REDIS_VERSION 4.0.2

# Install Postgres PHP extension dependencies
RUN apk --no-cache add postgresql-dev

# Install Redis PHP extension dependencies
RUN curl -L -o redis.tar.gz https://github.com/phpredis/phpredis/archive/$REDIS_VERSION.tar.gz && \
tar xfz redis.tar.gz && \
rm -r redis.tar.gz && \
mkdir -p /usr/src/php/ext && \
mv phpredis-* /usr/src/php/ext/redis

# Install Postgres and Redis PHP extensions
RUN docker-php-ext-install pdo_pgsql redis && docker-php-source delete

# Install Supervisor
RUN apk --no-cache add supervisor && mkdir /etc/supervisor.d

# Create user and group
RUN addgroup -S -g 1000 www && adduser -S -D -u 1000 -G www www

# Create workdir
RUN mkdir /www && touch /www/docker-volume-not-mounted && chown www:www /www
WORKDIR /www

# Supervisor will run PHP-FPM and Laravel queue workers
CMD ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
