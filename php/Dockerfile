FROM php:fpm-alpine

# Install Oracle Instant Client (required by Oci8 PHP extension)
ADD instantclient /tmp/instantclient
RUN echo "Installing Oracle Instant Client" && \
cd /tmp/instantclient && \
unzip instantclient-basic-linux.x64-12.1.0.2.0.zip && \
unzip instantclient-sdk-linux.x64-12.1.0.2.0.zip && \
mv instantclient_12_1 /usr/local/instantclient && \
ln -s /usr/local/instantclient/libclntsh.so.* /usr/local/instantclient/libclntsh.so && \
echo "Installing Oracle Instant Client dependencies" && \
mv glibc/sgerrand.rsa.pub /etc/apk/keys/ && \
apk --no-cache add libaio libnsl glibc/glibc-2.28-r0.apk && \
ln -s /usr/lib/libnsl.so.2 /usr/lib/libnsl.so.1

# Install Oci8 PHP extension
RUN docker-php-ext-configure oci8 --with-oci8=instantclient,/usr/local/instantclient && \
docker-php-ext-install oci8 && \
rm -rf /tmp/instantclient

# Create user and group
RUN addgroup -S -g 1000 www && adduser -S -D -u 1000 -G www www

# Create workdir
RUN mkdir /www && touch /www/docker-volume-not-mounted && chown www:www /www
WORKDIR /www

# Install Supervisor, who will run and monitor both PHP-FPM and Laravel queue workers
RUN apk --no-cache add supervisor && mkdir /etc/supervisor.d
CMD ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
