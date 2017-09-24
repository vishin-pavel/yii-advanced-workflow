FROM php:7-fpm as prod

MAINTAINER it.vishin@gmail.com

ENV PATH $PATH:/root/.composer/vendor/bin

# PHP extensions come first, as they are less likely to change between Yii releases
RUN \
       apt-get update \
    && apt-get install -y wget \
    && echo "deb http://nginx.org/packages/debian/ jessie nginx" >> /etc/apt/sources.list.d/nginx.list \
    && echo "deb-src http://nginx.org/packages/debian/ jessie nginx" >> /etc/apt/sources.list.d/nginx.list \
    && wget -q http://nginx.org/keys/nginx_signing.key \
    && apt-key add nginx_signing.key \
    && apt-get update \
    && apt-get -y install \
            g++ \
            libicu-dev \
            libmcrypt-dev \
            zlib1g-dev \
            nginx \
            supervisor \

    # Install PHP extensions
    && docker-php-ext-install intl \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install mcrypt \
    && docker-php-ext-install opcache \
    && docker-php-ext-install zip \
    && pecl install apcu-5.1.8 && echo extension=apcu.so > /usr/local/etc/php/conf.d/apcu.ini \

    && apt-get purge -y g++ \
    && apt-get autoremove -y \
    && rm -r /var/lib/apt/lists/* \

    # Don't clear our valuable environment vars in PHP
    && echo "\nclear_env = no" >> /usr/local/etc/php-fpm.conf \

    # Fix write permissions with shared folders
    && usermod -u 1000 www-data

COPY ./env/app/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY src /var/www/html
COPY ./env/app/sites/default.conf /etc/nginx/conf.d/default.conf

WORKDIR /var/www/html

CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]

FROM prod as dev

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer global require --no-progress "fxp/composer-asset-plugin:~1.3.1" \
    && composer global require --no-progress "codeception/codeception=2.0.*" \
    && composer global require --no-progress "codeception/specify=*" \
    && composer global require --no-progress "codeception/verify=*"

# Install XDebug
RUN yes | pecl install xdebug \
        && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
        && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
        && echo "xdebug.remote_autostart=off" >> /usr/local/etc/php/conf.d/xdebug.ini

# Install nodeJs + NPM
RUN curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
RUN chmod +x nodesource_setup.sh
RUN ./nodesource_setup.sh
RUN rm nodesource_setup.sh

RUN apt-get update \
    && apt-get -y install \
            nodejs \
            build-essential \
            git \
        --no-install-recommends \
    && rm -r /var/lib/apt/lists/*