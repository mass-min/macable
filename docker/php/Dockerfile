# NPMパッケージの事前ビルド
FROM node:14.17-alpine AS nodejs
ENV APP_ROOT /usr/src/macable

RUN mkdir -p $APP_ROOT
WORKDIR $APP_ROOT

COPY package.json package-lock.json* $APP_ROOT/

RUN apk add --no-cache git && \
    npm install --no-progress

COPY . $APP_ROOT
RUN npm run production

# PHPコンテナの作成
FROM php:7.4-fpm-alpine
ENV APP_ROOT /usr/src/macable
ENV COMPOSER_ALLOW_SUPERUSER 1
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN mkdir -p $APP_ROOT
WORKDIR $APP_ROOT

COPY composer.json composer.lock* $APP_ROOT/
COPY docker/php/www.conf /usr/local/etc/php-fpm.d/www.conf

SHELL ["/bin/sh", "-o", "pipefail", "-c"]
RUN apk update && \
    apk add --no-cache git mysql-client php7-sockets tzdata bash && \
    docker-php-ext-install -j$(nproc) pdo_mysql sockets && \
    cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
    echo "Asia/Tokyo" > /etc/timezone && \
    apk del tzdata && \
    echo decorate_workers_output=no >> /usr/local/etc/php-fpm.conf && \
    composer install --no-interaction --no-autoloader --no-scripts

COPY --from=nodejs $APP_ROOT $APP_ROOT
COPY --from=nodejs /usr/local/bin/node /usr/local/bin/
COPY --from=nodejs /usr/local/lib/ /usr/local/lib/
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
RUN ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

RUN composer dump-autoload --optimize

VOLUME $APP_ROOT/public/assets

ENTRYPOINT ["php-fpm", "-R"]
