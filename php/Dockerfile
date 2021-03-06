FROM php:fpm

RUN apt-get update && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        fontconfig-config fonts-dejavu-core wget locales \
    && docker-php-ext-install -j$(nproc) iconv mcrypt exif \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-install -j$(nproc) mysqli pdo_mysql
# php-pear #

RUN mkdir /php && chown www-data:www-data /php && chmod 770 /php

# echo listen = [::]:9000 \
RUN { \
      echo [global] ;\
      echo daemonize = no ;\
      echo ;\
      echo [www] ;\
      echo listen = /php/php.sock ;\
      echo listen.owner = www-data ;\
      echo listen.group = www-data ;\
      echo listen.mode = 0660 ;\
      echo slowlog = /proc/self/fd/2 ;\
      echo request_slowlog_timeout = 10s ;\
      echo security.limit_extensions = .php ;\
      echo rlimit_files = 10240 ;\
      echo php_value[output_buffering] = Off ;\
      echo php_admin_value[cgi.fix_pathinfo] = 0 ;\
      echo php_admin_value[upload_max_filesize] = 2G ;\
      echo php_admin_value[post_max_size] = 2G ;\
      echo php_admin_value[session.save_path] = /tmp ;\
      echo php_admin_value[pdo_mysql.default_socket] = /run/mysqld/mysqld.sock ;\
      echo php_admin_value[mysql.default_socket] = /run/mysqld/mysqld.sock ;\
      echo php_admin_value[mysqli.default_socket] = /run/mysqld/mysqld.sock ;\
    } | tee /usr/local/etc/php-fpm.d/zz-docker.conf

###########################
# Pydio
ARG PYDIO_VERSION=6.2.2
WORKDIR /srv
RUN wget http://downloads.sourceforge.net/project/ajaxplorer/pydio/stable-channel/${PYDIO_VERSION}/pydio-core-${PYDIO_VERSION}.tar.gz && \
    tar -zxf pydio-core-${PYDIO_VERSION}.tar.gz
RUN mv pydio-core-${PYDIO_VERSION} www && \
    mkdir /pydio-data && \
    mv /srv/www/data/files /pydio-data/files && \
    mv /srv/www/data/personal /pydio-data/personal && \
    mv /srv/www/data/logs /pydio-data/logs && \
    ln -s /pydio-data/files /srv/www/data/files && \
    ln -s /pydio-data/personal /srv/www/data/personal && \
    ln -s /pydio-data/logs /srv/www/data/logs && \
    chown -R www-data:www-data /srv/www /pydio-data

VOLUME /pydio-data

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG=en_US.UTF-8' > /etc/default/locale && \
    echo 'LANGUAGE=en_US:en' >> /etc/default/locale && \
    echo 'LC_ALL=en_US.UTF-8' >> /etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8


# RUN locale-gen en_US.UTF-8 && DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales

# NOTE: make sure to chown your data folder to 33:33

# RUN chmod -R 660 /srv/www
# RUN chmod 777 /srv/www/pydio-core/data/files/
# RUN chmod 777 /srv/www/pydio-core/data/personal/

CMD php-fpm