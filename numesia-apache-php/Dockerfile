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
telnet

#Ajout de igbinary (pour la serialisation de memcached)
RUN pecl install igbinary && echo "extension=igbinary.so" > "$PHP_INI_DIR/conf.d/igbinary.ini" 

#Ajout de memcached
#RUN printf "no --disable-memcached-sasl\n" | pecl install memcached \
#&& echo "extension=memcached.so" > "$PHP_INI_DIR/conf.d/memcached.ini"
RUN pecl install --onlyreqdeps --nobuild memcached \
&& cd "$(pecl config-get temp_dir)/memcached" \
&& phpize \
&& ./configure --enable-memcached-igbinary --disable-memcached-sasl \
&& make && make install \
&& echo "extension=memcached.so" > "$PHP_INI_DIR/conf.d/memcached.ini" \
&& echo "memcached.serializer=igbinary" >> "$PHP_INI_DIR/conf.d/memcached.ini"

#Ajout de PDO & MySQLi
RUN docker-php-ext-install pdo pdo_mysql mysqli calendar mbstring mcrypt intl pcntl zip soap sockets \
 && docker-php-ext-enable mysqli

#Changement de timezone
RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime \
 && echo "memory_limit=-1" > "$PHP_INI_DIR/conf.d/memory-limit.ini" \
 && echo "date.timezone=Europe/Paris" > "$PHP_INI_DIR/conf.d/date_timezone.ini"

#Install de composer
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV COMPOSER_VERSION 1.3.3

RUN curl -s -f -L -o /tmp/installer.php https://raw.githubusercontent.com/composer/getcomposer.org/da290238de6d63faace0343efbdd5aa9354332c5/web/installer
RUN php -r " \
    \$signature = '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410'; \
    \$hash = hash('SHA384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
        unlink('/tmp/installer.php'); \
        echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
        exit(1); \
    }" 
 RUN php /tmp/installer.php --no-ansi --install-dir=/usr/bin --filename=composer --version=${COMPOSER_VERSION}
 RUN rm /tmp/installer.php
 RUN composer --ansi --version --no-interaction

 #Install de Node
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash && apt-get install -y nodejs

#Install de Gulp
RUN npm install -g gulp
