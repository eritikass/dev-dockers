FROM php:7.1-apache as builder

RUN apt-get update \
	&& apt-get install -y \
		libmarkdown2-dev \
		apache2-dev \
		autoconf \
		libtool \
		automake \
		git

RUN mkdir -p /artifacts

### build mod_markdown

#COPY apache-mod-markdown /tmp/apache-mod-markdown
RUN git clone https://github.com/eritikass/apache-mod-markdown /tmp/apache-mod-markdown

RUN cd /tmp/apache-mod-markdown \
	&& autoreconf -f -i \
	&& ./configure --with-apxs=/usr/bin/apxs2 --with-discount=/usr \
	&& make \
	&& cp ./.libs/mod_markdown.so /artifacts/mod_markdown.so

## build mod_xsendfile

RUN git clone https://github.com/eritikass/mod_xsendfile /tmp/apache-mod_xsendfile \
	&& cd /tmp/apache-mod_xsendfile \
	&& apxs -cia mod_xsendfile.c \
	&& cp /usr/lib/apache2/modules/mod_xsendfile.so /artifacts/mod_xsendfile.so

######## base start ########

FROM php:7.1-apache
LABEL maintainer="eritikass@gmail.com"

#RUN mkdir -p /usr/lib/apache2/modules /etc/apache2/mods-enabled /etc/apache2/mods-available

### copy stuff
COPY --from=builder /artifacts/mod_markdown.so /usr/lib/apache2/modules/mod_markdown.so
COPY --from=builder /artifacts/mod_xsendfile.so /usr/lib/apache2/modules/xsendfile_module.so

RUN set -xe \
	# enable mods
	&& a2enmod rewrite \
	&& a2enmod actions \
	&& a2enmod cgi \
	&& a2enmod proxy \
    && a2enmod headers \
    && a2enmod alias \
	# read my custom configs
	&& echo "# my project configs" >> "/etc/apache2/apache2.conf" \
	&& echo "Include /etc/apache2/my-dev/*.conf" >> "/etc/apache2/apache2.conf" \
	&& mkdir -p /etc/apache2/my-dev && touch /etc/apache2/my-dev/test.conf \
	# install required packages
	&& apt-get update \
	&& apt-get install -y \
		vim \
		mysql-client \
		wget curl \
		git \
		subversion \
		cvs \
		locales \
		libssl-dev libsasl2-dev libcurl4-openssl-dev libxml2-dev zlib1g-dev libicu-dev g++ libmagickwand-dev imagemagick libpng-dev libjpeg-dev libjpeg62-turbo-dev libfreetype6-dev libmcrypt-dev libpng12-dev \
		# libmarkdown2 - required by mod_markdown
		libmarkdown2 \
	# install php extensions \
	&& docker-php-ext-install \
        #mysql \
		mysqli \
		pdo \
		pdo_mysql \
		iconv \
		mcrypt \
		pcntl \
		json \
		zip \
		curl \
		mbstring \
		soap \
		# spl \
		dom \
		xml \
		sockets \
		intl \
    # gd need some extra care
	&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    # install php pecl extensions \
	&& pecl install \
        #translit-0.6.1 \
        xdebug \
        imagick \
        solr \
        mongodb \
    && docker-php-ext-enable \
        #translit \
        xdebug \
        imagick \
        solr \
        mongodb \
	# move apache modules
	&& cd /etc/apache2/mods-enabled \
	&& ls -l /usr/lib/apache2/modules \
	# mod_markdown
    && echo "LoadModule markdown_module /usr/lib/apache2/modules/mod_markdown.so" > ../mods-available/mod_markdown.load \
    && ln -s ../mods-available/mod_markdown.load mod_markdown.load \
    # xsendfile_module
    && echo "LoadModule xsendfile_module /usr/lib/apache2/modules/xsendfile_module.so" > ../mods-available/xsendfile_module.load \
    && ln -s ../mods-available/xsendfile_module.load xsendfile_module.load \
	# enable et_EE locale \
	&& echo "et_EE.UTF-8 UTF-8" > /etc/locale.gen \
	&& locale-gen et_EE.UTF-8 \
	&& dpkg-reconfigure locales \
	# remove *-dev packages that were needed for compiling php extensions\
	&& apt-get remove -y \
		libssl-dev libsasl2-dev libcurl4-openssl-dev libxml2-dev zlib1g-dev libicu-dev g++ libmagickwand-dev imagemagick libpng-dev libjpeg-dev libjpeg62-turbo-dev libfreetype6-dev libmcrypt-dev libpng12-dev \
	# remove tmp files that were used only during build \
	&& apt-get clean \
	&& rm -rfv /var/lib/apt/lists/* /tmp/* /var/tmp/* \
