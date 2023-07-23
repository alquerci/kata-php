FROM php:8.2.4-cli-alpine

# Install composer as package manager
#
RUN set -eux; \
    prefix='/usr/local'; \
    buildDeps=' \
        curl \
    '; \
    apk add --no-cache \
        $buildDeps \
    ; \
    \
    curl -sSLf https://getcomposer.org/installer | php -- \
        --install-dir=${prefix}/bin \
        --filename=composer \
        --version=2.5.5 \
    ; \
    \
    mv ${prefix}/bin/composer ${prefix}/bin/composer.dist; \
    { \
      echo '#!/bin/sh'; \
      echo ''; \
      echo 'php -dmemory_limit=-1 '"${prefix}/bin/composer.dist"' ''"$@"'; \
    } | tee $prefix/bin/composer; \
    chmod +x $prefix/bin/composer; \
    \
    composer --version; \
    :;

# Install xdebug for phpunit code coverage
#
RUN set -xe; \
    buildDeps=' \
        autoconf \
        automake \
        build-base \
        linux-headers \
    '; \
    apk add --no-cache \
        $buildDeps \
    ; \
    \
    pecl install xdebug-3.2.1; \
    rm -rf /tmp/pear; \
    apk del $buildDeps; \
    :

# Install phpunit with xdebug is installed above.
#
RUN set -ex; \
    prefix=/usr/local; \
    { \
        echo '#!/bin/sh'; \
        echo ''; \
        echo 'enableXdebug='; \
        echo 'dashdash='; \
        echo ''; \
        echo 'for operand'; \
        echo 'do'; \
        echo '    case ${dashdash}${operand} in #('; \
        echo '        --)'; \
        echo '            dashdash='yes''; \
        echo '            ;; #('; \
        echo '        --coverage-*)'; \
        echo '            enableXdebug="-dzend_extension=xdebug.so -dxdebug.mode=coverage"'; \
        echo '            ;; #('; \
        echo '        *)'; \
        echo '            ;;'; \
        echo '    esac'; \
        echo 'done'; \
        echo ''; \
        echo 'php $enableXdebug /app/vendor/bin/phpunit ''"$@"'; \
    } | tee $prefix/bin/phpunit; \
    chmod +x $prefix/bin/phpunit; \
    :;

# Install infection with xdebug is installed above.
#
RUN set -ex; \
    prefix=/usr/local; \
    { \
        echo '#!/bin/sh'; \
        echo ''; \
        echo 'enableXdebug="-dzend_extension=xdebug.so -dxdebug.mode=coverage"'; \
        echo ''; \
        echo '/app/vendor/bin/infection --initial-tests-php-options="${enableXdebug}"' '"$@"'; \
    } | tee $prefix/bin/infection; \
    chmod +x $prefix/bin/infection; \
    :;

RUN docker-php-ext-install -j`nproc` pcntl
