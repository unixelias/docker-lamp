FROM httpd

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


COPY Dockerfile /root/buildinfo
COPY ./scripts/debconf.selections /tmp/
RUN debconf-set-selections /tmp/debconf.selections

# Set languages and locales
RUN apt-get update && apt-get upgrade -y && apt-get install -y locales tzdata \
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



# Install pre dependencies and set locales
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    bzip2 \
    curl \
    git \
    locales \
	openssl \	
	perl \
	nano \
    tzdata \
    unzip \
    xz-utils \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Apache + PHP requires preforking Apache for best results & enable Apache SSL
# forward request and error logs to docker log collector

# Install Build deps and PHP external modules
RUN echo "deb http://packages.dotdeb.org stretch all" >> /etc/apt/sources.list \
	&& echo "deb-src http://packages.dotdeb.org stretch all" >> /etc/apt/sources.list \
	&& curl -L http://www.dotdeb.org/dotdeb.gpg -o dotdeb.gpg \
	&& cat dotdeb.gpg | apt-key add - \
	&& apt-get update \
	&& apt-get upgrade -y \
	&& apt-get install -y --no-install-recommends \
		ftp \
		php \
		libapache2-mod-php \
		postfix \
		php-bz2 \
		php-cgi \
		php-cli \
		php-common \
		php-curl \
		php-dev \
		php-enchant \
		php-fpm \
		php-gd \
		php-gmp \
		php-imap \
		php-interbase \
		php-intl \
		php-json \
		php-ldap \
		php-mcrypt \
		php-mbstring \
		php-mysql \
		php-odbc \
		php-opcache \
		php-pgsql \
		php-phpdbg \
		php-pspell \
		php-readline \
		php-recode \
		php-snmp \
		php-sqlite3 \
		php-sybase \
		php-tidy \
		php-xmlrpc \
		php-xsl \
		php-zip \
		tree \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*

# Install npm and Yarn
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash \
	&& apt-get update && apt-get install -y --no-install-recommends npm \
  	&& apt-get clean && rm -rf /var/lib/apt/lists/* \
	&& npm install -g grunt-cli getcomposer	\
	&& curl -o- -L https://yarnpkg.com/install.sh | bash \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*


ENV LOG_STDOUT true
ENV LOG_STDERR true
ENV LOG_LEVEL warn
ENV ALLOW_OVERRIDE All
ENV DATE_TIMEZONE='America/Sao_Paulo'
ENV TERM dumb

COPY ./scripts/index.php /var/www/html/
COPY ./scripts/run-lamp.sh /usr/sbin/
COPY ./root/ ~/

RUN a2enmod mpm_prefork \
            ssl \
            rewrite \
    && a2ensite default-ssl \
    && ln -sf /dev/stdout /var/log/apache2/access.log \
	&& ln -sf /dev/stderr /var/log/apache2/error.log
	
RUN chmod +x /usr/sbin/run-lamp.sh
RUN chown -R www-data:www-data /var/www/html

VOLUME /var/www/html
VOLUME /var/log/httpd

EXPOSE 80
EXPOSE 443

CMD ["/usr/sbin/run-lamp.sh"]