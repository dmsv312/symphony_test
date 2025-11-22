FROM php:8.2-fpm

# Install dependencies for extensions and tools
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       git \
       unzip \
       libicu-dev \
       libzip-dev \
       libpng-dev \
       libjpeg-dev \
       libfreetype6-dev \
       libonig-dev \
       libxml2-dev \
       libwebp-dev \
       libxpm-dev \
       libsodium-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp --with-xpm \
    && docker-php-ext-install -j$(nproc) intl pdo_mysql gd zip opcache sodium \
    && docker-php-ext-enable opcache sodium \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');"

WORKDIR /var/www/shopware

CMD ["php-fpm"]
