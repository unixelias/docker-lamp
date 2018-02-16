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
	Usage="docker run -d -p [HOST HTTP PORT NUMBER]:80 -p [HOST HTTPS PORT NUMBER]:443 -v [HOST WWW DOCUMENT ROOT]:/var/www/html -v [HOST LOG FOLDER]:/var/log/httpd unixelias/lamp" \
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
	org.label-schema.docker.cmd="docker run -d -p 80:80 -p 443:443 -v /var/www/html:/var/www/html -v /var/log/httpd:/var/log/httpd unixelias/lamp"

ENV LOG_STDOUT='true' LOG_STDERR='true' LOG_LEVEL='warn' ALLOW_OVERRIDE='All'
ENV DATE_TIMEZONE='America/Sao_Paulo'
ENV TERM dumb

COPY Dockerfile /root/buildinfo
COPY ./scripts/debconf.selections /tmp/
RUN	debconf-set-selections /tmp/debconf.selections
COPY ./scripts/index.php /var/www/html/
COPY ./scripts/run-lamp.sh /usr/sbin/
COPY ./root/.banner ~/.banner
COPY ./keys/server.crt /etc/apache2/ssl/server.crt
COPY ./keys/server.key /etc/apache2/ssl/server.key
COPY ./scripts/dev.conf /etc/apache2/sites-enabled/dev.conf

# Add custom configs
RUN chmod +x /usr/sbin/run-lamp.sh \
	&& chown -R www-data:www-data /var/www/html \
	&& echo "# Display banner " >> ~/.bashrc \
	&& echo "if [ -f "$HOME/.banner" ]; then " >> ~/.bashrc \
	&& echo "	cat "$HOME/.banner" " >> ~/.bashrc \
	&& echo "fi " >> ~/.bashrc 

# Set languages and locales
RUN apt-get update && apt-get install -y locales tzdata \
	&& touch /usr/share/locale/locale.alias \
    && localedef -i pt_BR -c -f UTF-8 -A /usr/share/locale/locale.alias pt_BR.UTF-8 \
	&& echo "LC_ALL=pt_BR.UTF-8" >> /etc/environment \
	&& echo "pt_BR.UTF-8 UTF-8" >> /etc/locale.gen \
	&& echo "LANG=pt_BR.UTF-8" > /etc/locale.conf \
	&& echo "LC_ALL=en_US.UTF-8" >> /etc/environment \
	&& echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& echo "LANG=en_US.UTF-8" > /etc/locale.conf \
    && echo "portuguese	pt_BR" >> /etc/locale.alias \
	&& echo "portuguese	pt_BR.UTF-8" >> /etc/locale.alias \
	&& echo "english	en_US" >> /etc/locale.alias \
	&& echo "english	en_US.UTF-8" >> /etc/locale.alias \
	&& locale-gen pt_BR pt_BR.UTF-8 en_US en_US.UTF-8 \
	&& dpkg-reconfigure -f noninteractive locales \
    && unlink /etc/localtime \
    && ln -s /usr/share/zoneinfo/Brazil/East /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*
	
ENV LANG='pt_BR.UTF-8' LANGUAGE='pt_BR.UTF-8' LC_ALL='pt_BR.UTF-8'

# Install run deps
RUN runDeps=" \
		build-essential \
		bzip2 \
		ftp \
		git \
		gnupg2 \
		libfbclient2 \
		locales \
		mysql-client \
		openssl \
		perl \
		postfix \
		unzip \
		vim \
		xz-utils \
		tree \
		" \
	&& apt-get update \
    && apt-get install -y --no-install-recommends $runDeps \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Build deps and PHP external modules
RUN buildDeps=" \
	firebird-dev \
	libib-util \
	libldb-dev \
	libldap2-dev \
	libedit-dev \
	libreadline-dev \
	librecode-dev \
	libtidy-dev \
	libiodbc2-dev \
	libxslt1-dev \
	libbz2-dev \
	libc-client2007e-dev \
	libcurl4-openssl-dev \
	libenchant-dev \
	libmcrypt-dev \
	libfreetype6-dev \
	libgmp-dev \
	libjpeg62-turbo-dev \
	libkrb5-dev \
	libpng-dev \
	libssl-dev \
	libxml2-dev \
	zlib1g-dev \
	" \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends $buildDeps \
	&& docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
	&& docker-php-ext-install \
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
		mbstring \
		odbc \
		opcache \
		pcntl \
		pdo \
		pdo_mysql \
		phar \
		readline \
		recode \
		session \
		soap \
		tidy \
		xmlrpc \
		xsl \
		zip \
	&& docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
	&& docker-php-ext-install ldap \
	&& pecl install mcrypt-1.0.1 \
	&& docker-php-ext-enable mcrypt \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
	&& SUDO_FORCE_REMOVE=yes apt-get purge -y --auto-remove $buildDeps \
	&& apt-get clean && rm -rf /tmp && rm -rf /var/lib/apt/lists/* \
	&& a2enmod rewrite \
	&& a2enmod ssl \
	&& a2enmod mcrypt \
	&& service apache2 restart

# Install npm and Yarn
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash \
	&& apt-get update \
	&& apt-get upgrade -y \
	&& apt-get install -y --no-install-recommends npm \
  	&& apt-get clean && rm -rf /var/lib/apt/lists/* \
	&& npm install -g grunt-cli getcomposer	

RUN curl -o- -L https://yarnpkg.com/install.sh | bash \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*

VOLUME /var/www/html
VOLUME /var/log/httpd

EXPOSE 80
EXPOSE 443

CMD ["/usr/sbin/run-lamp.sh"]