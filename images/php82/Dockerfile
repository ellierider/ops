FROM --platform=$BUILDPLATFORM php:8.2.0-apache-buster

MAINTAINER Imarc <info@imarc.com>

# fix for postgresql-client
RUN mkdir -p /usr/share/man/man1 && \
    mkdir -p /usr/share/man/man7

# install supporting packages
RUN apt-get update && apt-get install -y --fix-missing \
    apt-transport-https \
    autoconf \
    build-essential \
    chrpath \
    curl \
    dnsutils \
    ffmpeg \
    freetds-dev \
    g++ \
    git-core \
    gnupg \
    imagemagick \
    iproute2 \
    libcurl4-openssl-dev \
    libdbd-freetds \
    libjpeg62-turbo-dev \
    libldap2-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libmemcached-dev \
    libmemcached11 \
    libpng-dev \
    libpq-dev \
    libpspell-dev \
    libsqlite3-dev \
    libwebp-dev \
    libzip-dev \
    default-mysql-client \
    netcat \
    net-tools \
    pkg-config \
    postgresql-client \
    python \
    unixodbc-dev \
    vim \
    wget

RUN curl -sSL https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN curl -sSL https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list

RUN apt-get update

RUN ACCEPT_EULA=y apt-get install -y \
    msodbcsql17 \
    mssql-tools

# link libsysbdb libs. (required for sqlsrv extension)
RUN ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so
RUN ln -s /usr/lib/x86_64-linux-gnu/libsybdb.a /usr/lib/libsybdb.a

# Configure GD with jpeg
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp

# Configure LDAP
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/

# Enable Exif
RUN docker-php-ext-configure exif --enable-exif

# install officially supported php extensions
RUN docker-php-ext-install \
    bcmath \
    curl \
    exif \
    gd \
    iconv \
    intl \
    ldap \
    mysqli \
    opcache \
    pcntl \
    pdo \
    pdo_dblib \
    pdo_mysql \
    pdo_pgsql \
    pdo_sqlite \
    pgsql \
    pspell \
    soap \
    zip

# install pecl extensions
RUN pecl install \
    memcached \
    imagick \
    redis \
    xdebug-3.2.0RC2 \
    sqlsrv \
    pdo_sqlsrv

# enable pecl extensions
RUN docker-php-ext-enable \
    memcached \
    redis \
    xdebug \
    sqlsrv \
    pdo_sqlsrv

# cleanup apt
RUN apt-get clean
RUN apt-get autoremove -y
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/src/*

# install composer
WORKDIR /tmp
RUN wget https://getcomposer.org/composer.phar
RUN mv composer.phar /bin/composer
RUN chmod 755 /bin/composer

# enable apache modules
RUN a2enmod \
    cache \
    cache_disk \
    expires \
    headers \
    macro \
    http2 \
    lua \
    rewrite \
    vhost_alias

# copy php.ini
COPY php.ini /usr/local/etc/php/conf.d

# copy apache config
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf

# set working directory
WORKDIR /var/www

# entrypoint/command
COPY docker-entrypoint /usr/local/bin/
RUN chmod 755 /usr/local/bin/docker-entrypoint
ENTRYPOINT ["/usr/local/bin/docker-entrypoint"]
CMD ["apache2-foreground"]
