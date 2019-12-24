# FROM php:7.4-fpm
# RUN apt-get update && apt-get install -y \
#     curl \
#     wget \
#     git \
#     libonig-dev \
#     libfreetype6-dev \
#     libjpeg62-turbo-dev \
#     libmcrypt-dev \
#     libpng-dev \
#     && docker-php-ext-install -j$(nproc) iconv mysqli pdo_mysql \
#     && docker-php-ext-configure gd --with-jpeg=/usr/include/ --with-freetype=/usr/include/ \
#     && pecl install mcrypt-1.0.2 \
#     && docker-php-ext-enable mcrypt \
#     && docker-php-ext-install -j$(nproc) gd
FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    curl \
    wget \
    git \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ADD php.ini /usr/local/etc/php/conf.d/40-custom.ini
WORKDIR /var/www
CMD ["php-fpm"]