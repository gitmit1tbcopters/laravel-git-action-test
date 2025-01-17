FROM php:8.4-cli

ARG WWWGROUP
WORKDIR /var/www/html

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
ENV SUPERVISOR_PHP_COMMAND="/usr/local/bin/php -d variables_order=EGPCS /var/www/html/artisan serve --host=0.0.0.0 --port=80"
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
    libpng-dev libjpeg-dev libfreetype6-dev libzip-dev zip unzip \
    libcurl4-openssl-dev pkg-config libssl-dev xvfb libicu-dev netcat-openbsd \
    supervisor net-tools libc-client-dev libkrb5-dev libldap2-dev libxml2-dev libcap2-bin && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Configure and install PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) gd zip pdo_mysql intl bcmath soap ldap && \
    pecl install imap && \
    docker-php-ext-enable imap

# Install Composer
COPY --from=composer:2.8.4 /usr/bin/composer /usr/bin/composer

# Install NVM and Node.js
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash && \
    export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"


# Copy configuration and setup files
COPY ./start-container /usr/local/bin/start-container
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./php.ini /usr/local/etc/php/conf.d/99-sail.ini

# Ensure necessary scripts are executable
RUN chmod +x /usr/local/bin/start-container

# Expose port
EXPOSE 80/tcp

ENTRYPOINT ["start-container"]
