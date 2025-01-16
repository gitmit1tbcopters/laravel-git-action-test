FROM php:8.4-cli-alpine

ARG WWWGROUP
ARG NODE_VERSION=22
ARG MYSQL_CLIENT="mysql-client"
WORKDIR /var/www/html

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
ENV SUPERVISOR_PHP_COMMAND="/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan serve --host=0.0.0.0 --port=80"
ENV SUPERVISOR_PHP_USER="sail"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99custom

# Install system dependencies and PHP extensions
RUN apk add --no-cache \
    gd curl imap mysql-client libpng-dev libjpeg-turbo-dev libfreetype6-dev libzip-dev libcap zip unzip git \
    libcurl4-openssl-dev pkgconfig libssl-dev wget xvfb libicu-dev openssh-server passwd supervisor nano \
    net-tools netcat-openbsd rsync bash && \
    docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) gd zip pdo_mysql json intl bcmath soap readline ldap \
    msgpack igbinary redis swoole memcached pcov imagick xdebug

# Clean up
RUN apk del libpng-dev libjpeg-turbo-dev libfreetype6-dev libzip-dev libicu-dev && \
    apk autoremove --purge && \
    rm -rf /var/cache/apk/*

# Install NVM
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash \
    && export NVM_DIR="$HOME/.nvm" \
    && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"


# Sail setup
RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.4

RUN groupadd --force -g ${WWWGROUP} sail
RUN useradd -ms /bin/bash --no-user-group -g ${WWWGROUP} -u 1337 sail

COPY ../start-container /usr/local/bin/start-container
COPY ../supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ../php.ini /etc/php/8.4/cli/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

# Configure SSH
COPY sshd_config /etc/ssh/
# Copy setup_ide.sh script into the container
COPY setup_ide.sh /usr/local/bin/setup_ide.sh

EXPOSE 80/tcp

ENTRYPOINT ["start-container"]
