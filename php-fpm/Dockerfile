#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

ARG PHP_VERSION
FROM expocker/php-fpm:${PHP_VERSION}

LABEL maintainer="Artem Fufaldin <artem.fufaldin@gmail.com>"

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

ARG PHP_VERSION

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update && \
    pecl channel-update pecl.php.net && \
    apt-get -o DPkg::options::="--force-confold" -y upgrade
    

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_SOAP=true
#

###########################################################################
# GD:
###########################################################################

ARG INSTALL_GD=false
RUN if [ ${INSTALL_GD} = true ]; then \
  apt-get install -y libjpeg-dev \
                     libpng-dev \
                     libfreetype6-dev && \
  docker-php-ext-configure gd --prefix=/usr \
                              --with-jpeg \
                              --with-freetype && \
  docker-php-ext-install gd && \
  php -r 'var_dump(gd_info());' \
;fi

###########################################################################
# ZIP:
###########################################################################

ARG INSTALL_ZIP=false
RUN if [ ${INSTALL_ZIP} = true ]; then \
  apt-get -y install libzip-dev zip unzip && \
  docker-php-ext-configure zip && \
  # Install the zip extension
  docker-php-ext-install zip && \
  php -m | grep -q 'zip' \
;fi

###########################################################################
# BZ2:
###########################################################################

ARG INSTALL_BZ2=false
RUN if [ ${INSTALL_BZ2} = true ]; then \
  apt-get -y install libbz2-dev; \
  docker-php-ext-install bz2 \
;fi

###########################################################################
# SOAP:
###########################################################################

ARG INSTALL_SOAP=false

RUN if [ ${INSTALL_SOAP} = true ]; then \
    # Install the soap extension
    rm /etc/apt/preferences.d/no-debian-php && \
    apt-get -y install libxml2-dev php-soap && \
    docker-php-ext-install soap \
;fi

###########################################################################
# MYSQL:
###########################################################################

ARG INSTALL_MYSQL=true

RUN if [ ${INSTALL_MYSQL} = true ]; then \
    apt-get install -y libpq-dev && \
    # Install the pdo_pgsql pgsql extensions
    docker-php-ext-install pdo_mysql mysql \
;fi

###########################################################################
# PGSQL:
###########################################################################

ARG INSTALL_PGSQL=true

RUN if [ ${INSTALL_PGSQL} = true ]; then \
    apt-get install -y libpq-dev && \
    # Install the pdo_pgsql pgsql extensions
    docker-php-ext-install pdo_pgsql pgsql \
;fi

###########################################################################
# xDebug:
###########################################################################

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
  # Install the xdebug extension
  pecl install xdebug && \
  docker-php-ext-enable xdebug \
;fi

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# Image optimizers:
###########################################################################

USER root

ARG INSTALL_IMAGE_OPTIMIZERS=false

RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get install -y jpegoptim optipng pngquant gifsicle \
;fi

###########################################################################
# ImageMagick:
###########################################################################

USER root

ARG INSTALL_IMAGEMAGICK=false

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y libmagickwand-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick \
;fi

###########################################################################
# Install additional locales:
###########################################################################

ARG INSTALL_ADDITIONAL_LOCALES=true
ARG ADDITIONAL_LOCALES

RUN if [ ${INSTALL_ADDITIONAL_LOCALES} = true ]; then \
    apt-get install -y locales \
    && echo '' >> /usr/share/locale/locale.alias \
    && temp="${ADDITIONAL_LOCALES%\"}" \
    && temp="${temp#\"}" \
    && for i in ${temp}; do sed -i "/$i/s/^#//g" /etc/locale.gen; done \
    && locale-gen \
;fi

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

ARG TZ=UTC

RUN echo "date.timezone = ${TZ}" >> /usr/local/etc/php/conf.d/timezone.ini

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

# Configure locale.
ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
