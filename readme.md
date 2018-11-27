# Laravel Docker Stack

Skeleton for creating a multi-container [Docker](https://www.docker.com) application based on [Laravel](https://laravel.com) framework.

> For use in development environment only. ***Not suitable for use in production environment!***.

The stack is created using [Docker Compose](https://docs.docker.com/compose/) and consist of the following services:

- [Nginx](https://nginx.org/en/) HTTP server.
- [PHP FPM](https://php-fpm.org) FastCGI process manager.
- [PostgreSQL](https://www.postgresql.org) relational database.
- [Redis](https://redis.io) in-memory cache and message broker.

## Features

- Containers are based on [Alpine Linux](https://alpinelinux.org) which makes them slim and fast.
- Source code and configuration files are mounted via Docker volumes so changes are reflected immediately, without having to rebuild the containers.
- Database persisted data is also mounted via Docker volume so changes are not lost after rebuilding the containers.
- Dependencies are managed using [PHP Composer](https://getcomposer.org) official Docker image to keep all the tools inside Docker land.
- All services are run by UID:GID `1000:1000` to prevent permission problems.

## Requirement

- [Git](https://git-scm.com).
- [Docker](https://www.docker.com) and [Docker Compose](https://docs.docker.com/compose/).

To prevent permission problems ensure all files and directories are owned by user:group `1000:1000`.

## Installation

Get the application skeleton

	git clone git@github.com:Stolz/laravel-docker-stack.git my-laravel-project && cd my-laravel-project

Install your Laravel flavor

	git clone git@github.com:laravel/laravel
	docker run --rm -it --user 1000:1000 -v $(pwd)/laravel:/app -v $(pwd)/composer:/tmp composer install

Start the services

	docker-compose up

## Configuration

Configure environment following [standard Laravel instructions](https://laravel.com/docs/master/configuration):

	cp laravel/.env.example laravel/.env
	$EDITOR laravel/.env

To use Postgres database with the default user set these values ...

	DB_CONNECTION=pgsql
	DB_HOST=postgres
	DB_DATABASE=postgres
	DB_USERNAME=postgres
	DB_PASSWORD=secret

... but since `postgres` is a privileged account you should consider creating a dedicated user instead. If you do so, don't forget to also update the service configuration in `docker-compose.yml` file.

To use Redis, first you need to install [Predis](https://github.com/nrk/predis) client ...

	docker run --rm -it --user 1000:1000 -v $(pwd)/laravel:/app -v $(pwd)/composer:/tmp composer require predis/predis

... and then you can use these values

	BROADCAST_DRIVER=redis
	CACHE_DRIVER=redis
	QUEUE_CONNECTION=redis
	SESSION_DRIVER=redis
	REDIS_HOST=redis

## Artisan

You can run Artisan directly from the host. For instance

	docker exec -it php ./artisan key:generate
	docker exec -it php ./artisan migrate

If you prefer, you can connect to the container and run Artisan from inside it

	docker exec -it php ash
	./artisan --help

## Queue

[Supervisor](http://supervisord.org) is used to automatically run Laravel queue workers. You can [customize the configuration](https://laravel.com/docs/master/queues#supervisor-configuration) by modifying `php/supervisord.conf` file.

## TODO

- Add Nodejs to the stack to compile Laravel assets.
