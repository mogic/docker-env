FROM php:7.2-fpm

# install phpredis extension
ENV PHPREDIS_VERSION 4.2.0

# Install modules
RUN apt-get update \
    && docker-php-ext-install json pcntl pdo pdo_mysql mysqli mbstring opcache tokenizer

# Install bcmath extension for rabbitmq client
RUN docker-php-ext-configure bcmath \
    && docker-php-ext-install bcmath

# Install intl extension
RUN apt-get install -y libicu-dev \
    && docker-php-ext-install intl

# Install sockets extension for php
RUN docker-php-ext-install sockets

# Install gd extension
RUN apt-get update \
    && apt-get install -y libfreetype6-dev libgd-dev libjpeg62-turbo-dev  libpng-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd

# Install geoip  extension
RUN apt-get install -y libgeoip-dev \
    && pecl install http://pecl.php.net/get/geoip-1.1.1.tgz \
    && docker-php-ext-enable geoip

# Install php-redis  extension
RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && ( \
        cd phpredis-$PHPREDIS_VERSION \
        && phpize \
        && ./configure \
        && make \
        && make install \
    ) \
    && rm -r phpredis-$PHPREDIS_VERSION \
    && docker-php-ext-enable redis

# Install imagick  extension
RUN apt-get install -y libmagickwand-dev --no-install-recommends && rm -rf /var/lib/apt/lists/* \
    && pecl install imagick \
    && docker-php-ext-enable imagick

# Install curl & zip extension
RUN apt-get update && apt-get install -y zlib1g-dev \
    && docker-php-ext-install zip

# Enable and configure xdebug
# Docker for Mac (2016-08-22): need to run on host: sudo ifconfig lo0 alias 10.254.254.254
# see https://forums.docker.com/t/ip-address-for-xdebug/10460/4
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "xdebug.max_nesting_level=400" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.remote_autostart=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.default_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.profiler_enable=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.remote_host=10.254.254.254" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_port=9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.idekey='PHPSTORM'" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	echo "xdebug.remote_log='/tmp/xdebug.log'" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini;

# Install composer
RUN apt-get update && apt-get install -y curl git
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer
RUN composer self-update

# Install blackfire
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
    && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini

CMD ["php-fpm"]