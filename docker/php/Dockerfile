FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    curl \
    wget \
    git \
    && docker-php-ext-install -j$(nproc) iconv mysqli pdo_mysql \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN docker-php-ext-install pdo pdo_mysql
ADD ./docker/php/php.ini /usr/local/etc/php/conf.d/40-custom.ini
WORKDIR /var/www
CMD ["php-fpm"]
