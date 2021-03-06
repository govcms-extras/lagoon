ARG CLI_IMAGE
ARG LAGOON_IMAGE_VERSION
ARG PHP_IMAGE_VERSION
ARG COMPOSER_AUTH
FROM ${CLI_IMAGE} as cli

FROM amazeeio/php:${PHP_IMAGE_VERSION}-cli-drupal-${LAGOON_IMAGE_VERSION}

ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz

COPY --from=cli /app /app
COPY --from=cli /usr/bin/phpunit /app/web/profiles/contrib/govcms/.docker/phpunit
COPY --from=cli /usr/bin/lint-govcms /app/web/profiles/contrib/govcms/.docker/scripts/lint-govcms

RUN cp -r /app/web/profiles/contrib/govcms/tests /app/ \
    && echo "memory_limit=-1" >> /usr/local/etc/php/conf.d/memory.ini \
    && composer install -d /app/tests -n --ansi --prefer-dist --no-suggest \
    && rm -rf /usr/local/etc/php/conf.d/memory.ini \
    && mv /app/web/profiles/contrib/govcms/.docker/scripts/lint-govcms /usr/bin/lint-govcms \
    && chmod +x /usr/bin/lint-govcms \
    && mv /app/web/profiles/contrib/govcms/.docker/scripts/lint-theme /usr/bin/lint-theme \
    && chmod +x /usr/bin/lint-theme \
    && mv /app/web/profiles/contrib/govcms/.docker/scripts/behat /usr/bin/behat \
    && chmod +x /usr/bin/behat \
    && mv /app/web/profiles/contrib/govcms/.docker/phpunit /usr/bin/phpunit \
    && chmod +x /usr/bin/phpunit

COPY images/test/9.x/scripts/drutiny /usr/bin/drutiny

COPY images/test/9.x/config/phpunit.xml /app/tests/phpunit/
COPY images/test/9.x/scripts/bootstrap.php /app/tests/phpunit/

# Ensure MySQL client can accept server max_allowed_packet
COPY --from=cli /etc/my.cnf.d /etc/my.cnf.d

ARG SITE_AUDIT_VERSION
RUN git clone --single-branch --branch=$SITE_AUDIT_VERSION https://github.com/govcms/audit-site.git /app/web/sites/all/drutiny \
    && php -d memory_limit=-1 /usr/local/bin/composer --working-dir=/app/web/sites/all/drutiny/ install --ignore-platform-reqs \
    && rm -Rf /app/web/sites/all/drutiny/.git \
    && chmod +x /usr/bin/drutiny

# Ensure that drush and drush.launcher both work
ENV WEBROOT=web

# Add custom drutiny profiles from test folder for local development
COPY images/test/9.x/config/*.profile.yml /app/web/sites/all/drutiny/Profiles
