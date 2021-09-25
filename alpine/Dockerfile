FROM alpine:3.12

MAINTAINER Stephen <admin@stephen520.cn>

ARG work_dir=/var/www/html

ENV PHP_VERSION=7.4.8 \
    PKG_CONFIG_VERSION=0.29.2 \
    PHPZIP_VERSION=1.19.0 \
    PHPREDIS_VERSION=5.3.1

# Libs
RUN apk update && apk add --no-cache make libc-dev gcc libxml2-dev openssl-dev sqlite-dev curl-dev oniguruma-dev autoconf libzip-dev libpng-dev \
	&& apk cache && apk del \
	&& mkdir -p ${work_dir} \

	&& cd /tmp/ \
	&& wget http://pkgconfig.freedesktop.org/releases/pkg-config-${PKG_CONFIG_VERSION}.tar.gz -O pkg-config.tar.gz \
	&& tar -zxvf pkg-config.tar.gz \
	&& cd pkg-config-${PKG_CONFIG_VERSION}/ \
	&& ./configure --with-internal-glib \
	&& make && make install \
	&& cd .. \
	&& rm -rf pkg-config-* \

	# user
	&& groupadd www-data && useradd -g www-data www-data

# Install PHP
RUN wget https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz -O php.tar.gz \
	&& tar -zxvf php.tar.gz \
	&& cd php-${PHP_VERSION}/ \
	&& ./configure  --prefix=/usr/local/php \
	--with-config-file-scan-dir=/usr/local/php/conf.d \
	--with-curl \
	--with-mysqli \
	--with-openssl \
	--with-pdo-mysql \
	--enable-gd \
	--enable-fpm \
	--with-fpm-user=www-data \
	--with-fpm-group=www-data \
	--enable-bcmath \
	--enable-xml \
	--enable-mbstring \
	--enable-pcntl \
	--enable-sockets \
	--enable-opcache \
	&& make && make install \
	&& export PATH=/usr/local/php/bin:/usr/local/php/sbin/:/etc/init.d/:$PATH \
	&& cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf \
	&& cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf \
	&& cp ./php.ini-production /usr/local/php/lib/php.ini \
	&& cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm \
	&& chmod a+x /etc/init.d/php-fpm \
	&& cd .. \
	&& rm -rf php-* \

	# Install zip extension
	&& wget http://pecl.php.net/get/zip \
	&& tar -zxvf zip \
	&& cd zip-${PHPZIP_VERSION}/ \
	&& phpize \
	&& ./configure --with-php-config=/usr/local/php/bin/php-config \
	&& make && make install \
	&& cd .. \
	&& rm -rf zip* \
	&& echo 'extension=zip.so' >> /usr/local/php/lib/php.ini \

	# Install redis extension
	&& wget https://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz -O redis.tgz \
	&& tar -zxvf redis.tgz \
	&& cd redis-${PHPREDIS_VERSION} \
	&& phpize \
	&& ./configure --with-php-config=/usr/local/php/bin/php-config \
	&& make && make install \
	&& cd .. \
	&& rm -rf redis-* \
	&& echo 'extension=redis.so' >> /usr/local/php/lib/php.ini

WORKDIR ${work_dir}

EXPOSE 9000

CMD ["/etc/init.d/php-fpm", "start"]
#CMD ["/usr/local/php/sbin/php-fpm", "-F"]
