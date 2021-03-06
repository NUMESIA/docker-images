FROM php:7.1-apache

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
default-mysql-client \
cron \
inetutils-ping \
libmemcached-dev memcached \
libfreetype6-dev \
libjpeg62-turbo-dev \
libjpeg-dev \
libpng-dev \
libssl-dev \
telnet \
locales

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
 
#Changement de locale
RUN sed -i 's/# fr_FR.UTF-8 UTF-8/fr_FR.UTF-8 UTF-8/g' /etc/locale.gen \
 && locale-gen fr_FR.UTF-8

ENV LANG fr_FR.UTF-8
ENV LANGUAGE fr_FR:fr
ENV LC_ALL fr_FR.UTF-8

#Install de composer
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
&& php composer-setup.php --quiet --no-ansi --install-dir=/usr/bin --filename=composer \
&& rm composer-setup.php \
&& composer --ansi --version --no-interaction

 #Install de nvm et node
ENV NVM_DIR /root/.nvm
ENV NODE_VERSION 8.16.0

RUN cd /root && mkdir /root/.nvm && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash


# ajout de node et npm au path
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# install de Gulp
RUN npm install -g gulp

#Compatibilité avec vieux TLS
RUN apt-get update -yqq \
    && apt-get install -y --no-install-recommends openssl \
    && sed -i 's,^\(MinProtocol[ ]*=\).*,\1'TLSv1.0',g' /etc/ssl/openssl.cnf \
    && sed -i 's,^\(CipherString[ ]*=\).*,\1'DEFAULT@SECLEVEL=1',g' /etc/ssl/openssl.cnf
