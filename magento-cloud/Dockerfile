FROM ubuntu:18.04
LABEL maintainer="bill@halfpricebanners.com"

# We make fairly extensive use of ARGs here so that the history of the docker file
# doesn't remember them, and also so that they can be overridden on the CLI if you
# so desire. Honestly, since you're running this container locally, the defaults
# (including credentials) should be fine.  However, if you want to change things,
# this ARGs section is where you should do it.

# Set Up Timezone - See https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
# for valid TZ strings
ARG TZ=America/Chicago
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Going forward, no end-user-defined variables exist. 
#
# Here there be dragons.

# Tell the system that there's no one that can tapitty-type
ARG DEBIAN_FRONTEND=noninteractive

# Install Necessary Packages

# System
RUN apt update
RUN apt install -y lsof
RUN apt install -y wget
RUN apt install -y sudo
RUN apt install -y vim
RUN apt install -y git
RUN apt install -y curl
RUN apt install -y htop
RUN apt install -y rsync
RUN apt install -y software-properties-common
RUN apt install -y default-jre-headless
RUN echo 'JAVA_HOME=/usr/lib/jvm/default-java' >> /etc/environment
RUN apt install -y apt-transport-https
RUN add-apt-repository -y ppa:chris-lea/redis-server
RUN wget -O - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
RUN add-apt-repository -y "deb https://artifacts.elastic.co/packages/6.x/apt stable main"
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash -
RUN apt update

# Services + PHP Mods
RUN apt install -y sendmail
RUN apt install -y mysql-server-5.7
RUN apt install -y nginx
RUN apt install -y php php-bcmath php-curl php-gd php-intl php-mbstring php-mysql php-soap php-xdebug php-xml php-zip
RUN apt install -y php-fpm
RUN apt install -y redis-server
RUN apt install -y rabbitmq-server
RUN apt install -y elasticsearch
RUN apt install -y socat
RUN apt install -y composer
RUN apt install -y nodejs
RUN npm install -g requirejs
RUN npm install -g xmlhttprequest
RUN apt install -y siege
RUN wget -O phpunit https://phar.phpunit.de/phpunit-8.phar && chmod +x phpunit && mv phpunit /usr/bin/

# Add User, grant sudo permissions
RUN useradd -m -s /bin/bash magentodev
RUN usermod -G sudo -a magentodev

# Copy in relevant config files
COPY ./etc/nginx/sites-available/magento-nginx.conf /etc/nginx/sites-available/
COPY ./etc/php/7.2/fpm/php.ini /etc/php/7.2/fpm/
COPY ./etc/php/7.2/fpm/pool.d/magento-fpm.conf /etc/php/7.2/fpm/pool.d/
COPY ./etc/php/7.2/mods-available/xdebug.ini /etc/php/7.2/mods-available/
COPY ./etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/
COPY ./etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/
COPY ./etc/sudoers /etc/

# Remove remnant default files, link up new config
RUN rm -f /etc/php/7.2/fpm/pool.d/www.conf
RUN rm -f /etc/nginx/sites-enabled/default
RUN ln -s /etc/nginx/sites-available/magento-nginx.conf /etc/nginx/sites-enabled/magento-nginx.conf

# Prep SQL service for Magento DB
RUN chown -R mysql:mysql /var/lib/mysql && /etc/init.d/mysql start && \
	mysql --defaults-file=/etc/mysql/debian.cnf \
    -e "CREATE DATABASE main; CREATE USER 'user'@'localhost'; GRANT ALL ON main.* TO 'user'@'localhost';"

# System work is finished. Let's get magento up and running
USER magentodev
WORKDIR /home/magentodev
RUN curl -sS https://accounts.magento.cloud/cli/installer | php

# Final tidbits
RUN sudo apt --purge autoremove -y ; sudo apt clean

# ENTRYPOINT bash

ENTRYPOINT sudo chown -R mysql:mysql /var/lib/mysql && \
	sudo rm /var/lib/mysql/mysql.sock || true &&  \
	sudo service mysql start && \
	sudo service sendmail start && \
	sudo service redis-server start && \
	sudo service php7.2-fpm start && \
	sudo service rabbitmq-server start && \
	sudo service nginx start && \
	sudo /etc/init.d/elasticsearch start && \
	socat tcp-l:6370,fork,reuseaddr tcp:127.0.0.1:6379

EXPOSE 80 3306 5672 6379 9200 15672