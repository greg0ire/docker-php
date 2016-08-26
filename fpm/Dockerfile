FROM php:7.0-fpm-alpine

MAINTAINER Arnaud Chevillard <achevillard@universcine.com>

RUN set -xe \
        && apk add --no-cache --virtual .phpext-builddeps \
            libbz2 \
            gettext-dev \
            libmcrypt-dev \
            libxslt-dev \
            zlib-dev \
            postgresql-dev \
            autoconf gcc libc-dev make \
            libpng-dev libjpeg-turbo-dev \
            icu-dev \
            libev \
            openldap-dev \

        && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
        && docker-php-ext-configure ldap --with-libdir=lib/ \
        && docker-php-ext-install \
            bcmath \
            bz2 \
            calendar \
            exif \
            ftp \
            gd \
            gettext \
            intl \
            ldap \
            mbstring \
            mcrypt \
            opcache \
            pcntl \
            pdo_mysql \
            pdo_pgsql \
            pspell \
            shmop \
            sockets \
            soap \
            sysvmsg \
            sysvsem \
            sysvshm \
            xsl \
            zip \

        && find /usr/local/lib/php/extensions -name '*.a' -delete \
        && find /usr/local/lib/php/extensions -name '*.so' -exec strip --strip-all '{}' \; \
        && runDeps="$( \
            scanelf --needed --nobanner --recursive \
            /usr/local/lib/php/extensions \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
        )" \
        && apk add --virtual .phpext-rundeps $runDeps \
        && apk del .phpext-builddeps \

        # Time Zone
        && echo "date.timezone=${PHP_TIMEZONE:-UTC}" > $PHP_INI_DIR/conf.d/date_timezone.ini \
        # Short open tags
        && echo "short_open_tag = Off" > $PHP_INI_DIR/conf.d/disable_short_open_tag.ini \

        # install composer
        && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
        && php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified';  } else { echo 'Installer corrupt'; unlink('composer-setup.php');  } echo PHP_EOL;" \
        && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
        && rm composer-setup.php \
        && chmod +x /usr/local/bin/composer \
        && composer self-update \

        # cleanup from apk
        && rm -rf /tmp/* /var/cache/apk/*

WORKDIR /srv

CMD ["php-fpm", "--nodaemonize"]
