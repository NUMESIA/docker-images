FROM php:7.0-apache

#Activation des modules apache
RUN a2enmod rewrite ssl proxy proxy_http headers

RUN apt-get update && apt-get install -y \
git \
subversion \
bash \
curl \
unzip \
vim \
supervisor \
bindfs \
libmcrypt-dev \
zlib1g-dev \
libicu-dev \
g++ \
gnupg \
libxml2-dev \
libfontconfig \
libxrender1 \
wkhtmltopdf \
mysql-client \
cron \
inetutils-ping \
libmemcached-dev memcached \
libfreetype6-dev \
libjpeg62-turbo-dev \
libjpeg-dev \
libpng-dev \
telnet

#Ajout de igbinary (pour la serialisation de memcached)
RUN pecl install igbinary && echo "extension=igbinary.so" > "$PHP_INI_DIR/conf.d/igbinary.ini" 

#Ajout de memcached
RUN pecl install --onlyreqdeps --nobuild memcached \
&& cd "$(pecl config-get temp_dir)/memcached" \
&& phpize \
&& ./configure --enable-memcached-igbinary --disable-memcached-sasl \
&& make && make install \
&& echo "extension=memcached.so" > "$PHP_INI_DIR/conf.d/memcached.ini" \
&& echo "memcached.serializer=igbinary" >> "$PHP_INI_DIR/conf.d/memcached.ini"

#Ajout de PDO & MySQLi
RUN docker-php-ext-install pdo pdo_mysql mysqli calendar mbstring mcrypt intl pcntl zip soap sockets \
 && docker-php-ext-enable mysqli \
 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
 && docker-php-ext-install -j$(nproc) gd

#Changement de timezone
RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime \
 && echo "memory_limit=-1" > "$PHP_INI_DIR/conf.d/memory-limit.ini" \
 && echo "date.timezone=Europe/Paris" > "$PHP_INI_DIR/conf.d/date_timezone.ini"

#Install de composer
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
&& php composer-setup.php --quiet --no-ansi --install-dir=/usr/bin --filename=composer \
&& rm composer-setup.php \
&& composer --ansi --version --no-interaction

 #Install de Node
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash && apt-get install -y nodejs

#Install de Gulp
RUN npm install -g gulp
