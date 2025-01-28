FROM php:8.4-fpm

ARG WWWGROUP

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
ENV SUPERVISOR_PHP_USER="sail"


# Set timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Configure apt
RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99custom

# Patch pearcmd.php to remove E_STRICT
RUN sed -i 's/E_STRICT/E_ALL/' /usr/local/lib/php/pearcmd.php

# Install system dependencies, excluding Imagick-specific libraries
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpng-dev libjpeg-dev libfreetype6-dev libzip-dev zip unzip git \
    libcurl4-openssl-dev pkg-config libssl-dev wget xvfb libicu-dev openssh-server netcat-openbsd \
    passwd supervisor nano net-tools rsync libc-client-dev libkrb5-dev libldap2-dev libxml2-dev libcap2-bin sudo && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Configure and install PHP extensions
# RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
#     docker-php-ext-install -j$(nproc) gd zip pdo_mysql intl bcmath soap ldap pcntl && \
#     pecl install imap swoole xdebug-3.4.1 && \
#     docker-php-ext-enable imap xdebug swoole

RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) gd zip pdo_mysql intl bcmath soap ldap && \
    pecl install imap xdebug-3.4.1 && \
    docker-php-ext-enable imap xdebug

# Install Composer
COPY --from=composer:2.8.4 /usr/bin/composer /usr/bin/composer

RUN echo $NVM_DIR

# Install prerequisites and NVM
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash && \
    export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
    echo "export NVM_DIR=$NVM_DIR" >> ~/.bashrc && \
    echo "[ -s \"$NVM_DIR/nvm.sh\" ] && . \"$NVM_DIR/nvm.sh\"" >> ~/.bashrc

RUN mkdir -p /var/run/sshd

# Set up Sail permissions
RUN setcap "cap_net_bind_service=+ep" /usr/local/bin/php
RUN groupadd --force -g ${WWWGROUP} sail
RUN useradd -ms /bin/bash --no-user-group -g ${WWWGROUP} -u 1337 sail

# Copy configuration and setup files
COPY ./start-container /usr/local/bin/start-container
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./php.ini /usr/local/etc/php/conf.d/99-sail.ini
#COPY ./setup_ide /usr/local/bin/setup_ide
COPY ./setup_ssh /usr/local/bin/setup_ssh
#COPY ./setup_vapor /usr/local/bin/setup_vapor
COPY ./sshd_config /etc/ssh/

# Ensure necessary scripts are executable
#RUN chmod +x /usr/local/bin/start-container /usr/local/bin/setup_ide
RUN chmod +x /usr/local/bin/start-container /usr/local/bin/setup_ssh

# Expose port
EXPOSE 80/tcp


ENTRYPOINT ["start-container"]
