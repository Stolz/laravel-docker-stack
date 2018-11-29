# Laravel Docker Stack

Skeleton for creating a multi-container [Docker](https://www.docker.com) application based on [Laravel](https://laravel.com) framework.

> For use in development environment only. ***Not suitable for use in production environment!***.

The stack is created using [Docker Compose](https://docs.docker.com/compose/) and consist of the following services:

- [Nginx](https://nginx.org/en/) HTTP server.
- [PHP FPM](https://php-fpm.org) FastCGI process manager.
- [Oracle](https://www.oracle.com/database/) relational database.
- [Redis](https://redis.io) in-memory cache and message broker.

## Features

- Containers (except database) are based on [Alpine Linux](https://alpinelinux.org) which makes them slim and fast.
- Database container based on the slim version of Oracle enterprise.
- Source code and configuration files are mounted via Docker volumes so changes are reflected immediately, without having to rebuild the containers.
- Database persisted data is also mounted via Docker volume so changes are not lost after rebuilding the containers.
- Dependencies are managed using [PHP Composer](https://getcomposer.org) official Docker image to keep all the tools inside Docker land.
- All services are run by UID:GID `1000:1000` to prevent permission problems.

## Requirement

- [Git](https://git-scm.com).
- [Docker](https://www.docker.com) and [Docker Compose](https://docs.docker.com/compose/).
- [Oracle Instant Client](https://www.oracle.com/technetwork/database/database-technologies/instant-client/downloads/index.html) installation files (.zip).
- A valid [Oracle Xontainer Registry](https://container-registry.oracle.com) account. You will need to accept the license available at `Database > enterprise` section.

## Installation

[Download Oracle Instant Client](https://www.oracle.com/technetwork/database/database-technologies/instant-client/downloads/index.html) ZIP files (basic + sdk) and place then inside `php/instantclient/` folder.

Log into your Oracle Container Registry account

	docker login -u your@email.com container-registry.oracle.com

Get the application skeleton

	git clone git@github.com:Stolz/laravel-docker-stack.git -b oracle my-laravel-project && cd my-laravel-project

Get glibc compatibility layer package for Alpine Linux

	wget -P php/instantclient/glibc/ https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.28-r0/glibc-2.28-r0.apk

Install your Laravel flavor

	git clone git@github.com:laravel/laravel
	docker run --rm -it --user 1000:1000 -v $(pwd)/laravel:/app -v $(pwd)/composer:/tmp composer install

Start the services

	docker-compose up

## Configuration

Configure environment following [standard Laravel instructions](https://laravel.com/docs/master/configuration):

	cp laravel/.env.example laravel/.env
	$EDITOR laravel/.env

Create a database schema ...

	docker exec -it oracle bash
	source /home/oracle/.bashrc
	alter session set container = database;
	create user <user> identified by <password> container = current;
	grant connect, resource, create any context to <user>;
	alter user <user> quota unlimited on users;

... and then you can use these values

	DB_CONNECTION=oracle
	DB_HOST=oracle
	DB_PORT=1521
	DB_SERVICE=DATABASE.DOCKER
	DB_USERNAME=<user>
	DB_PASSWORD=<password>

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
