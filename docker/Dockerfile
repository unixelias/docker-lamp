FROM php:7.2-apache

MAINTAINER Elias Alves <elias.alves@ufvjm.edu.br>

ARG BUILD_DATE
ARG VCS_REF
ARG VCS_BRANCH

LABEL Name="Development Web Server Stack" \
      Version=$VCS_BRANCH \
      Architecture="x86_64" \
			License="Apache License 2.0" \
      Dockerfile_location="/root/buildinfo" \
			Usage="docker run -d -p [HOST WWW PORT NUMBER]:80 -v [HOST WWW DOCUMENT ROOT]:/var/www/html -v [HOST DB DOCUMENT ROOT]:/var/lib/mysql unixelias/lamp" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="Development Web Server Stack" \
      org.label-schema.description="Cutting-edge Web stack, based on Ubuntu 16.04 LTS. Includes .htaccess support and popular PHP7 features, including composer and mail() function." \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/unixelias/docker-lamp" \
      org.label-schema.vendor="Elias Alves at UFVJM" \
      org.label-schema.version=$VCS_BRANCH \
      org.label-schema.version="0.1" \
      org.label-schema.version="dev" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.docker.cmd="docker run -d -p [HOST WWW PORT NUMBER]:80 -v [HOST WWW DOCUMENT ROOT]:/var/www/html -v [HOST DB DOCUMENT ROOT]:/var/lib/mysql unixelias/lamp"

ENV LOG_STDOUT='true' LOG_STDERR='true' LOG_LEVEL='warn' ALLOW_OVERRIDE='All'
ENV DATE_TIMEZONE='America/Sao_Paulo'
ENV TERM dumb
ENV LANG='pt_BR.UTF-8' LANGUAGE='pt_BR.UTF-8' LC_ALL='pt_BR.UTF-8'

COPY Dockerfile /root/buildinfo
COPY ./scripts/debconf.selections /tmp/
COPY ./scripts/index.php /var/www/html/
COPY ./scripts/run-lamp.sh /usr/sbin/
COPY ./root/.banner ~/.banner
COPY ./keys/server.crt /etc/apache2/ssl/server.crt
COPY ./keys/server.key /etc/apache2/ssl/server.key
COPY ./scripts/dev.conf /etc/apache2/sites-enabled/dev.conf
RUN debconf-set-selections /tmp/debconf.selections \
	&& chmod +x /usr/sbin/run-lamp.sh \
	&& chown -R www-data:www-data /var/www/html \
	&& echo "# Display banner " >> ~/.bashrc \
	&& echo "if [ -f "$HOME/.banner" ]; then " >> ~/.bashrc \
	&& echo "	cat "$HOME/.banner" " >> ~/.bashrc \
	&& echo "fi " >> ~/.bashrc 
	

# Install pre dependencies and set locales
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    build-essential \
	bzip2 \
    git \
	gnupg2 \
    locales \
	openssl \	
	perl \
    tzdata \
    unzip \
    vim \
    xz-utils \
    && locale-gen pt_BR pt_BR.UTF-8 en_US en_US.UTF-8 \
    && echo "portuguese	pt_BR" >> /etc/locale.alias \
	&& echo "portuguese	pt_BR.UTF-8" >> /etc/locale.alias \
	&& dpkg-reconfigure -f noninteractive locales \
	&& source ~/.bashrc \
    && unlink /etc/localtime \
    && ln -s /usr/share/zoneinfo/Brazil/East /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN curl -sL https://deb.nodesource.com/setup_8.x | bash \
	&& apt-get update \
	&& apt-get upgrade -y \
	&& apt-get install -y --no-install-recommends \
	ftp \
	libbz2-dev \
	libcurl4-openssl-dev \
	libmcrypt-dev \
	libfreetype6-dev \
	libjpeg62-turbo-dev \
	libpng-dev \
	libssl-dev \
	libxml2-dev \
	mysql-client \
	npm \
	pkg-config \
	postfix \
	tree \
	zlib1g-dev \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN npm install -g grunt-cli getcomposer	

RUN curl -o- -L https://yarnpkg.com/install.sh | bash

RUN docker-php-ext-install \
	bz2 \
	calendar \
	curl \
	dom \
	enchant \
	fileinfo \
	ftp \
	gd \
	gettext \
	gmp \
	hash \
	imap \
	interbase \
	intl \
	json \
	ldap \
	mbstring \
	odbc \
	opcache \
	pcntl \
	pdo \
	pdo_dblib \
	pdo_firebird \
	pdo_mysql \
	pdo_oci \
	pdo_odbc \
	pdo_pgsql \
	pdo_sqlite \
	pgsql \
	phar \
	readline \
	recode \
	session \
	soap \
	sockets \
	tidy \
	tokenizer \
	xml \
	xmlreader \
	xmlrpc \
	xmlwriter \
	xsl \
	zend_test \
	zip \
	&& apt-get clean && rm -rf /tmp && rm -rf /var/lib/apt/lists/*

RUN pecl install mcrypt-1.0.1 \
	&& docker-php-ext-enable mcrypt \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
RUN a2enmod rewrite \
	&& a2enmod ssl \
	&& a2enmod mcrypt \
	&& service apache2 restart

VOLUME /var/www/html
VOLUME /var/log/httpd

EXPOSE 80
EXPOSE 443

CMD ["/usr/sbin/run-lamp.sh"]