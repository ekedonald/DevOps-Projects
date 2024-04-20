# Use an official PHP image as a parent image
FROM php:7.4-cli

USER root

# ENV DB_HOST=
# ENV DB_DATABASE=
# ENV DB_USERNAME=
# ENV DB_PASSWORD=
ENV COMPOSER_ALLOW_SUPERUSER=1


# Set the working directory in the container
WORKDIR /var/www/html

# Install dependencies (including Composer)
RUN apt-get update && apt-get install -y     libpng-dev     zlib1g-dev     libxml2-dev     libzip-dev     libonig-dev     zip     curl     unzip     && docker-php-ext-configure gd     && docker-php-ext-install -j2 gd     && docker-php-ext-install pdo_mysql     && docker-php-ext-install mysqli     && docker-php-ext-install zip     && docker-php-source delete     && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Copy the rest of the application code into the container
COPY . /var/www/html

# Install Laravel dependencies
RUN composer install

# Expose port 8000 for the Laravel development server
EXPOSE 8000

# Define the command to start the Laravel development server
ENTRYPOINT [ "bash", "app.sh" ]
