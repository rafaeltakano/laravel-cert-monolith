FROM php:7.4.30-apache-buster

RUN apt-get update && apt-get upgrade -y

# debian sys dependencies
RUN apt-get install -y sudo nano

# debian sys dependencies for php extensions
RUN apt-get install -y libicu-dev libzip-dev libpng-dev

# debian sys dependencies for puppeteer
RUN apt-get install -y libxshmfence-dev \
  libgbm-dev \
  wget \
  unzip \
  fontconfig \
  locales \
  gconf-service \
  libasound2 \
  libatk1.0-0 \
  libc6 \
  libcairo2 \
  libcups2 \
  libdbus-1-3 \
  libexpat1 \
  libfontconfig1 \
  libgcc1 \
  libgconf-2-4 \
  libgdk-pixbuf2.0-0 \
  libglib2.0-0 \
  libgtk-3-0 \
  libnspr4 \
  libpango-1.0-0 \
  libpangocairo-1.0-0 \
  libstdc++6 \
  libx11-6 \
  libx11-xcb1 \
  libxcb1 \
  libxcomposite1 \
  libxcursor1 \
  libxdamage1 \
  libxext6 \
  libxfixes3 \
  libxi6 \
  libxrandr2 \
  libxrender1 \
  libxss1 \
  libxtst6 \
  ca-certificates \
  fonts-liberation \
  libappindicator1 \
  libnss3 \
  lsb-release \
  xdg-utils

# debian sys dependencies for wpp-connect-server
RUN wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
RUN sudo apt install -y ./google-chrome-stable_current_amd64.deb
RUN rm -rf ./google-chrome-stable_current_amd64.deb

# debian apt cache clear
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# install php extensions for laravel
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
RUN sed -ri 's/;date.timezone =/date.timezone = "America\/Sao_Paulo"/g' /usr/local/etc/php/php.ini
RUN docker-php-ext-install intl gd zip pdo_mysql
RUN pecl install xdebug-3.1.5 && docker-php-ext-enable xdebug

# install composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# install nodejs
RUN curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh
RUN bash nodesource_setup.sh
RUN apt-get install -y nodejs
RUN rm nodesource_setup.sh

# conf for apache2 - adoorei-checkout
ENV APACHE_DOCUMENT_ROOT=/var/www/html/adoorei-checkout/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN a2enmod rewrite headers

# conf for apache2 - adoorei-checkout w/ ssl
RUN a2enmod ssl && a2enmod socache_shmcb
RUN sed -ri '/SSLCertificateFile.*snakeoil\.pem/c\SSLCertificateFile \/etc\/ssl\/certs\/openssl.crt' /etc/apache2/sites-available/default-ssl.conf 
RUN sed -ri '/SSLCertificateKeyFile.*snakeoil\.key/cSSLCertificateKeyFile /etc/ssl/private/openssl.key\' /etc/apache2/sites-available/default-ssl.conf
RUN a2ensite default-ssl

# conf for apache2 - atendimento-wpp
COPY ./apache/001-default.conf /etc/apache2/sites-available/001-default.conf
COPY ./apache/apache2.conf /etc/apache2/apache2.conf
RUN a2ensite 001-default.conf

# create user adooreidev
RUN useradd -G www-data,root -u 1000 -d /home/adooreidev adooreidev
RUN mkdir -p /home/adooreidev/.composer && chown -R adooreidev:adooreidev /home/adooreidev
RUN echo adooreidev:adooreidev | chpasswd adooreidev
RUN usermod -aG sudo adooreidev
WORKDIR /var/www/html
USER adooreidev

# create localhost certificate
RUN echo adooreidev | sudo -S openssl req -x509 -new -out /etc/ssl/certs/openssl.crt -keyout /etc/ssl/private/openssl.key -days 365 -newkey rsa:4096 -sha256 -nodes \
  -subj "/C=BR/ST=Goias/L=Goiania/O=Adoorei/OU=IT/CN=adooreisub/emailAddress=rafael@adoorei.com"
RUN echo adooreidev | sudo -S chmod 777 -R /etc/ssl
