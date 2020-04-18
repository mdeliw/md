[toc]

# Docker

## Install Docker Desktop

Runs on MAC OS 10.12+.

```bash
sysctl kern.hv_support # must display kern.nv_support: 1
```

- [Download Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
- Launch docker.dmg
- Open Docker, Grant priviledges

```bash
docker version
docker-compose version
docker run hello-world
```

## Docker Commands

```bash
docker --version
docker info
docker run hello-world

# docker <context> <command> <image> <proess to run>
docker container run alpine echo "Hello World"
docker container run -d --name quotes alpine echo "Hello World"

# ping the loopback address for five times
docker container run centos ping -c 5 127.0.0.1

# listing container
docker container ls -h	# help
docker container ls     # list only running containers
docker container ls --all # list all containers
docker container ls -a	# list all containers
docker container ls -l 	# list the last container
docker container ls -q	# just list the IDs of containers
docker container ls | grep <name> | awk '{print $1}' # ID of the container

# delete container
docker container rm [container_id]
docker container rm [container_name]

docker container rm [-force or -f] [container_name] # force delete if container is running

docker container rm --force $(docker container ls [--all or -a] [--quiet or -q]) # delete all containers
docker container rm -f $(docker container ls -aq) # delete all containers running or not

# auto-restart https://docs.docker.com/engine/reference/run/#restart-policies---restart
# enable auto-restart across Docker reboots
docker run --restart=always

# enable auto-restart until Docker reboots
docker run --restart=unless-stopped

# disable auto-restart
docker update --restart=no [container_name]

# disable auto-restart for all containers
docker update --restart=no $(docker ps -a -q)

# start a container
docker container start [container_name]

# stop a container
docker container stop [container_name] 
docker container stop $(docker ps -a -q)

# bash into a running container
docker container run --interactive --tty [container_name]
docker exec -it [container_name] /bin/bash

# start and run in background, publish a port inside the container (80) to the local (8088). External:Internal
docker container run [--detach or -d] [--publish or -p] 8088:80 [image_name]
docker container run [--detach or -d] [--publish or -p] 8088:80 [--name or -n] [friendly_container_name] [image_name]
docker container run --env TARGET=google.com --env INTERVAL=5000 [image_name] # changes an environment variable inside container
docker container run --name [container_name] -d -p 8080:80 --network [network_name] [image_name] 

# list processes running in the container
docker container top [container_id]
docker container logs [container_id]
docker container inspect [container_id]
docker container inspect -f "{{json .State}}" [container_name] | jq # needs brew install jq on local
docker container stats [container_id]

# run a container, change it, commit it for reuse
docker container run -it --name cn [image_name]
# change something inside container
docker container commit cn cn2
docker container run cn2

# pull image
docker image pull [image_name]
docker image history [image_name]

# run a dockerfile from current directory
docker image build --tag [new_image_name] .

# view all images
docker image ls # image size are logical not actual
docker image ls 'startWith*'
docker images -a
docker images | grep "none"
docker system df # shows how much disk space docker is using

# delete orphaned image
docker system prune

# remove docker images
docker rmi [image_name]
docker rmi [image_name]:[tag_name]
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
docker rmi $(docker images | grep "none" | awk '/ / { print $3 }')

docker image rm -f $(docker image ls -f reference='diamol/*' -q)

# executing inside a running container
docker container exec -i -t <container-name> /bin/sh
docker container exec <container-name> ps
docker container exec -it -e MY_VAR="Hello World" <container-name> /bin/sh

# attaching to a running container
docker container attach <container-name>
docker run -d --name nginx -p 8080:80 nginx:alpine # -d for daemon
curl -4 localhost:8080
docker container attach nginx
# new terminal
for n in {1..10}; do curl -4 localhost:8080; done

# retrieving container logs
docker container logs <container-name>
docker container logs --tail 5 <container-name>
docker container logs --tail 5 --follow <container-name>
# apps inside container must log to STDOUT / STDERR and not into files
# docker uses a logger drive, default is json-file
docker container run --name test -it --log-driver none busybox sh -c 'for n in 1 2 3; do echo "hello $n"; done'

docker-compose stop
docker-compose rm
docker-compose ps
docker-compose down -v

```

<img src="/Users/manish/Library/Application Support/typora-user-images/image-20191105185201953.png" alt="image-20191105185201953" style="zoom:25%;" />



##  Use a web container and change the index.html

```bash
docker container run --detach --publish 8080:80 [web_container_name] # container_id = 86b...

docker container exec [container_id] ls /usr/local/apache2/htdocs/index.html # show file inside container

dockerc container cp [container_id]:[source_file] [container_id]:[target_file]
docker container cp [external_index.html] [container_id]:/usr/local/apache2/htdocs/index.html

```

### Creating images

1. interactively build a container and commit it into an image
2. create image from Dockerfile
3. Import image into system from a tarball

```bash
docker container run -it --name sample alpine /bin/sh
apk update && apk add iputils
ping 127.0.0.1

docker container ls -a | grep sample
docker container diff sample
docker container commit sample my-alpine
docker image ls
docker image history my-alpine

docker image build
docker image save -o
docker image load -i
docker image tag 
docker login
docker image push

```

### _Dockerfile_

```dockerfile
FROM python:2.7
RUN mkdir -p /app
WORKDIR /app
COPY ./requirements.txt /app/
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

```Dock
FROM alpine:latest
ENTRYPOINT ["ping"]
CMD ["8.8.8.8", "-c", "3"]
```

```bash
docker image build -t pinger .
docker image build -t pinger -f Dockerfile .
docker container run --rm -it pinger
docker container run --rm -it --entrypoint /bin/sh pinger
```

### # Stateful Container

#### # Docker Developer Environment (Medium example)

[How To Run Your Entire Development Environment in Docker Containers on macOS](https://medium.com/better-programming/php-how-to-run-your-entire-development-environment-in-docker-containers-on-macos-787784e94f9a)

[How to Run MySQL in a Docker Container on macOS with Persistent Local Data](https://medium.com/@crmcmullen/how-to-run-mysql-in-a-docker-container-on-macos-with-persistent-local-data-58b89aec496a)

```bash
mkdir ~/Sites
mkdir ~/Develop
cd ~/Develop
	mkdir docker_configs
	mkdir logs
	mkdir mysql_data
	mkdir docker_dev_env

cd ~/Develop/docker_configs
	mkdir mysql
	mkdir redis

cd ~/Develop/logs
	mkdir apache
	mkdir php
```

### _Create a Redis config file_

[Redis Docker Image](https://hub.docker.com/_/redis/)

Create a  configuration file for the Redis server. We are going to allow the server in our development environment to listen on all IP address ports on the Docker network.  So comment the default `bind`  statement.

```bash
nano redis/redis_conf
```

```
# bind 127.0.0.1
protected-mode yes
```

### _Create a MySQL config file_

Add a single setting to run using native password authentication instead of the default `caching_sha2_password`.

```bash
nano mysql/my.cnf
```

```
[mysqld]
default-authentication-plugin=mysql_native_password
```

### _Create a PHP landing page_

```bash
echo "<?php phpinfo();" > ~/Sites/index.php
```

Disable the default apache that comes pre-installed with macOS. 

```bash
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/
```

### _Create a Docker Network_

```bash
docker network create dev-network
# 7916ec996f842a21d83c0e7672a1dd66839a5d3d414c464e0439c2474b60a9e1
```

### _Launch a Redis Server Container_

[Redis Docker Image](https://hub.docker.com/_/redis/)

We are using our own `redis.conf`, and to do that, we can use the command below. The alternative way is to create a  `Dockerfile`. 

- The `—-restart always`  will restart this container anytime Docker is started such as on laptop reboot or if Docker gets shutdown and started again. Leave this parameter out if you want to start your own containers every time.
- The `--name ` assigns name to the container instance.
- The `--network` will join this container to the docker network we created.
- The `-v` will bind the config file inside the container to the config file we provided.
- You can use `-d redis` for using the latest version.

```bash
docker run --restart always --name redis-localhost --net dev-network -v /Users/manish/Develop/docker_configs/redis/redis.conf:/usr/local/etc/redis/redis.conf -d redis

docker ps
```

#### _Launch a MySQL Server Container_

[MySQL Docker Image](https://hub.docker.com/_/mysql)

- You are binding the `/var/lib/mysql` folder inside the container to your `mysql_data/8.0` folder to provide your persistent data after container reboots.
- You are binding the config file `/etc/mysql/conf` inside the container to the config file we provided.

```bash
docker run --restart always --name mysql-localhost --net dev-network -v /Users/manish/Develop/mysql_data/8.0:/var/lib/mysql -v /Users/manish/Develop/docker_configs/mysql:/etc/mysql/conf.d -p 3306:3306 -d -e MYSQL_ROOT_PASSWORD=dmdone23 mysql:8.0

docker ps
```

#### _Launch an Apache / PHP Server Container_

We will create a [Dockerfile](https://gist.githubusercontent.com/crmcmullen/52e93b07f7ccb0d02abf95bbaac86ae4/raw/d82c5cecc4bb2e666bff4cedec3581982d7e08f9/Dockerfile) to build a custom image containing:

- [PHP with Apache](https://hub.docker.com/_/php/)
- [Xdebug](https://xdebug.org/) - PHP extension to assit in debugging and development
- [Igbinary](https://github.com/igbinary/igbinary) - Replacement for php serializer
- Redis PHP extension from [PECL](https://pecl.php.net/package/redis)

```bash
code ~/Develop/docker_dev_env/Dockerfile
```

```dockerfile
FROM php:7.2-apache

# Creating a PHP, MySQL and Redis development environment on macOS.
# This Dockerfile will create an APACHE, PHP 7.2 server that includes the Xdebug, Igbinary and 
#   Redis PHP extensions from PECL. It will also create PHP.ini overrides that will point session management 
#   to the Redis server created in this same article.
#

# run non-interactive. Suppresses prompts and just accepts defaults automatically.
ENV DEBIAN_FRONTEND=noninteractive

# update OS and install utils
RUN apt-get update; \
    apt-get -yq upgrade; \
    apt-get install -y --no-install-recommends \
    apt-utils \
    nano; \
    apt-get -yq autoremove; \
    apt-get clean; \ 
    rm -rf /var/lib/apt/lists/*

# make sure custom log directories exist
RUN mkdir /usr/local/log; \
    mkdir /usr/local/log/apache2; \
    mkdir /usr/local/log/php; \
    chmod -R ug+w /usr/local/log

# create official PHP.ini file
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini

# install MySQLi
RUN docker-php-ext-install mysqli 

# update PECL and install xdebug, igbinary and redis w/ igbinary enabled
RUN pecl channel-update pecl.php.net; \
    pecl install xdebug-2.7.2; \
    pecl install igbinary-3.0.1; \
    pecl bundle redis-5.0.2 && cd redis && phpize && ./configure --enable-redis-igbinary && make && make install; \
    docker-php-ext-enable xdebug igbinary redis

# Delete the resulting ini files created by the PECL install commands
RUN rm -rf /usr/local/etc/php/conf.d/docker-php-ext-igbinary.ini; \
    rm -rf /usr/local/etc/php/conf.d/docker-php-ext-redis.ini; \ 
    rm -rf /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Add PHP config file to conf.d
RUN { \
        echo 'short_open_tag = Off'; \
        echo 'expose_php = Off'; \    
        echo 'error_reporting = E_ALL & ~E_STRICT'; \
        echo 'display_errors = On'; \
        echo 'error_log = /usr/local/log/php/php_errors.log'; \
        echo 'upload_tmp_dir = /tmp/'; \
        echo 'allow_url_fopen = on'; \
        echo '[xdebug]'; \
        echo 'zend_extension="xdebug.so"'; \
        echo 'xdebug.remote_enable = 1'; \
        echo 'xdebug.remote_port = 9001'; \
        echo 'xdebug.remote_autostart = 1'; \
        echo 'xdebug.remote_connect_back = 0'; \
        echo 'xdebug.remote_host = host.docker.internal'; \
        echo 'xdebug.idekey = VSCODE'; \
        echo '[redis]'; \
        echo 'extension="igbinary.so"'; \
        echo 'extension="redis.so"'; \
        echo 'session.save_handler = "redis"'; \
        echo 'session.save_path = "tcp://redis-localhost:6379?weight=1&timeout=2.5"'; \
    } > /usr/local/etc/php/conf.d/php-config.ini

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /usr/local/log/apache2

# Configure apache mods
RUN a2enmod rewrite 

# Add ServerName parameter
RUN echo "ServerName localhost" | tee /etc/apache2/conf-available/servername.conf
RUN a2enconf servername

# Update the default apache site with the config we created.
RUN { \
        echo '<VirtualHost *:80>'; \
        echo '    ServerAdmin your_email@example.com'; \
        echo '    DocumentRoot /var/www/html'; \
        echo '    <Directory /var/www/html/>'; \
        echo '        Options Indexes FollowSymLinks MultiViews'; \
        echo '        AllowOverride All'; \
        echo '        Order deny,allow'; \
        echo '        Allow from all'; \
        echo '    </Directory>'; \
        echo '    ErrorLog /usr/local/log/apache2/error.log'; \
        echo '    CustomLog /usr/local/log/apache2/access.log combined' ; \
        echo '</VirtualHost>'; \
    } > /etc/apache2/sites-enabled/000-default.conf

EXPOSE 80
```

```bash
cd ~/Develop/docker_dev_env
docker build -t dev-environment .
docker images -a
```

#### _Launch the Development Environment Image into a Container_

```bash
docker run --restart always --name www-localhost --net dev-network -v /Users/manish/Sites:/var/www/html -v /Users/manish/Develop/logs:/usr/local/log -p 80:80 -d dev-environment
```

Now open a browser and navigate to localhost. The `index.php` file we created earlier should load. 