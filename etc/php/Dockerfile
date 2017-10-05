FROM php:7.1-fpm

MAINTAINER Lukasz Malicki <lukasz.malicki@polcode.net>

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng12-dev \
    zlib1g-dev \
    libicu-dev \
    jpegoptim \
    g++ \
    libxrender1 \
    libfontconfig \
    nodejs-legacy \
    npm \
    nano \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install intl \
    && docker-php-ext-install zip

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

RUN npm install -g gulp bower

# Set timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
RUN "date"

# Type docker-php-ext-install to see available extensions
RUN docker-php-ext-install pdo pdo_mysql

# install xdebug
# xdebug and PHP config in php.ini
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug

RUN echo 'alias sf="php app/console"' >> ~/.bashrc
RUN echo 'alias sf3="php bin/console"' >> ~/.bashrc
RUN echo 'alias lv="php artisan"' >> ~/.bashrc

WORKDIR /var/www/api
