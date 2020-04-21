## Motivation
There are tons of examples of using Docker with PHP-projects combined with ready for development Docker-images. But I didn't find any examples of projects using PHP with Docker setup both for Development and Production simultaneously.

Most of the examples out there are using Docker Volumes. Instead of packing all required project code into Image - it is mounting into Container at runtime.

From other side - copying full project code into Image is not an option during development as you want to see changes of the code without rebuilding the Image.

This Repo is an example of how such problems can be solved: by using custom Dockefiles and Docker Compose for Nginx and PHP-FPM it is capable to build an Image which contains all required software to run your Laravel-project in single Docker Container (assuming your Database is a separate service)

This solution based on next projects: 
- [http://laradock.io](http://laradock.io)
- [https://github.com/richarvey/nginx-php-fpm](https://github.com/richarvey/nginx-php-fpm)

## Overview
Docker images are located in [.docker-laravel](https://github.com/rodion-arr/laravel-docker-example/tree/master/.docker-laravel) folder. There are 2 docker-compose files (dev and prod) for building 3 containers:
- `mysql` - MySQL server for local development
- `workspace` - a Laradock utility container with Composer, Node and MySQL client for local development so you don't need all this stuff installed on you machine. 
- `nginx-php-fpm` - web-server with copied Laravel code
  - Image build process:
    - Stage 1:
        - Based on laradock/workspace Image
        - install project dependencies (composer and npm)
        - Run PHPUnit tests
        - Run Frontend tests
        - Build frontend scaffolding
        - Prepare files for Stage 2 (clean-up dev dependencies, logs, node_modules)
    - Stage 2 (web-server, PHP)
        - Based on richarvey/nginx-php-fpm Image
        - PHP 7.4
        - Nginx
        - Copy only required for project run-time files from Stage 1 (reduces Image size)

## Prerequisites
Testes on latest Docker Desktop on Mac (2.2.0.5)

## Usage #1 (local development)
Clone project and set-up env file
```bash
git clone https://github.com/rodion-arr/laravel-docker-example.git
cd laravel-docker-example
cp .env.example .env
```
Run workspace container and install dependencies
```bash
cd .docker-laravel
docker-compose up -d workspace
docker-compose exec workspace bash
composer install
npm install
php artisan key:generate
exit
```
Build and run web-server container
```
docker-compose build nginx-php-fpm
docker-compose up -d
```
Open browser and [http://localhost](http://localhost). Container bootstraps up to 20 seconds.

## Usage #2 (packing project code in one independent Image)
Clone project and set-up env file
```bash
git clone https://github.com/rodion-arr/laravel-docker-example.git
cd laravel-docker-example
cp .env.example .env
```
Build and run web-server container. It will install all dependencies and run tests.
```bash
cd .docker-laravel
docker-compose build nginx-php-fpm
docker-compose -f docker-compose.prod.yml up -d
```
Open browser and [http://localhost](http://localhost). Container bootstraps up to 20 seconds.

## License
[MIT license](https://opensource.org/licenses/MIT).
