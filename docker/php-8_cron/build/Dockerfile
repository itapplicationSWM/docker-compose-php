FROM php:8.0-fpm-alpine
RUN apk add --no-cache mysql-client msmtp perl wget procps shadow libzip libpng libjpeg-turbo libwebp freetype icu icu-data-full

RUN apk add --no-cache --virtual build-essentials \
    icu-dev icu-libs zlib-dev g++ make automake autoconf libzip-dev \
    libpng-dev libwebp-dev libjpeg-turbo-dev freetype-dev && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install gd && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install pdo_mysql && \
    docker-php-ext-install intl && \
    docker-php-ext-install opcache && \
    docker-php-ext-install exif && \
    docker-php-ext-install zip && \
    apk del build-essentials && rm -rf /usr/src/php*

RUN wget https://getcomposer.org/composer-stable.phar -O /usr/local/bin/composer && chmod +x /usr/local/bin/composer

RUN usermod -u 1050 www-data && groupmod -g 1050 www-data

#FROM php:8.0-fpm
#RUN apt-get update && apt-get install -y \
#		libfreetype-dev \
#		libjpeg62-turbo-dev \
#		libpng-dev \
#	&& docker-php-ext-configure gd --with-freetype --with-jpeg \
#	&& docker-php-ext-install -j$(nproc) gd

RUN apk add \
    unixodbc \
    unixodbc-dev

RUN apk add m4
RUN apk add make
RUN apk add build-base

RUN curl http://ftp.gnu.org/gnu/autoconf/autoconf-latest.tar.gz > autoconf.tar.gz && \
    tar -xvzf autoconf.tar.gz && \
    cd autoconf-2.71/ && \
    ./configure && make && make install && export PHP_AUTOCONF=/usr/local/bin/autoconf


RUN pecl install sqlsrv \
    && pecl install pdo_sqlsrv

RUN docker-php-ext-enable sqlsrv \
    && docker-php-ext-enable pdo_sqlsrv


RUN apk --no-cache add bash

RUN ARCH=$(uname -m) && \
    case "$ARCH" in \
        x86_64) architecture="amd64" ;; \
        arm64) architecture="arm64" ;; \
        *) architecture="unsupported" ;; \
    esac && \
    if [ "unsupported" == "$architecture" ]; then \
        echo "Alpine architecture $ARCH is not currently supported." && exit 1; \
    fi && \
    curl -O https://download.microsoft.com/download/3/5/5/355d7943-a338-41a7-858d-53b259ea33f5/msodbcsql18_18.3.2.1-1_$architecture.apk && \
    curl -O https://download.microsoft.com/download/3/5/5/355d7943-a338-41a7-858d-53b259ea33f5/mssql-tools18_18.3.1.1-1_$architecture.apk && \
    apk add gnupg && \
    curl -O https://download.microsoft.com/download/3/5/5/355d7943-a338-41a7-858d-53b259ea33f5/msodbcsql18_18.3.2.1-1_$architecture.sig && \
    curl -O https://download.microsoft.com/download/3/5/5/355d7943-a338-41a7-858d-53b259ea33f5/mssql-tools18_18.3.1.1-1_$architecture.sig && \
    curl https://packages.microsoft.com/keys/microsoft.asc  | gpg --import - && \
    gpg --verify msodbcsql18_18.3.2.1-1_$architecture.sig msodbcsql18_18.3.2.1-1_$architecture.apk && \
    gpg --verify mssql-tools18_18.3.1.1-1_$architecture.sig mssql-tools18_18.3.1.1-1_$architecture.apk && \
    apk add --allow-untrusted msodbcsql18_18.3.2.1-1_$architecture.apk && \
    apk add --allow-untrusted mssql-tools18_18.3.1.1-1_$architecture.apk


RUN apk add nano
#RUN apk add openrc --no-cache
RUN apk add --update --no-cache autoconf g++ imagemagick-dev libtool make pcre-dev \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && apk del autoconf g++ libtool make pcre-dev \
    && apk add php81-pecl-imagick

#RUN apk update && apk add busybox-extras

RUN echo "* * * * * cd /srv/project/etichette.test && php artisan schedule:run >> /dev/null 2>&1" >> /etc/crontabs/root

RUN apk add busybox-initscripts
RUN rc-update add crond
RUN export EDITOR=/usr/bin/nano
CMD ["crond", "-f"]
