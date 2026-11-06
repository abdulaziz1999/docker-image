FROM webdevops/php-nginx:8.2

ENV DEBIAN_FRONTEND=noninteractive

COPY conf/ /opt/docker/

# add nginx key
RUN curl -s https://nginx.org/keys/nginx_signing.key | apt-key add -

# apt update
RUN apt-get update -q

# Asia/Jakarta Timezone
RUN ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime

# Locale id_ID
RUN apt-get install -qq locales
RUN sed -i -e 's/# id_ID.UTF-8 UTF-8/id_ID.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure locales && \
    update-locale LANG=id_ID.UTF-8
ENV LANG=id_ID.UTF-8

# upgrade openssl
RUN apt-get install -qq --no-install-recommends openssl \
    && sed -i 's,^\(MinProtocol[ ]*=\).*,\1'TLSv1.0',g' /etc/ssl/openssl.cnf \
    && sed -i 's,^\(CipherString[ ]*=\).*,\1'DEFAULT@SECLEVEL=1',g' /etc/ssl/openssl.cnf

# install composer
ENV COMPOSER_ALLOW_SUPERUSER=1
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# install wkhtmltopdf
RUN apt-get install -qq \
    xvfb \
    xauth \
    xfonts-base \
    xfonts-75dpi \
    fontconfig \
    libssl3 \
    libjpeg62-turbo \
    libxrender1 \
    wget

# install tools
RUN apt-get install -qq \
    iproute2 \
    git \
    zip

# install sqlsrv pdo_sqlsrv
RUN apt-get install -qq \
    apt-transport-https \
    gnupg2 \
    && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update -qq

RUN ACCEPT_EULA=Y apt-get install -qq \
    msodbcsql17 \
    libgss3 \
    odbcinst \
    unixodbc \
    unixodbc-dev

RUN pecl install \
    sqlsrv-5.12.0 \
    pdo_sqlsrv-5.12.0

RUN docker-php-ext-enable \
    sqlsrv \
    pdo_sqlsrv

# oci8
RUN apt-get install libaio1 unzip -qq
COPY instantclient/ /tmp/instantclient
RUN cd /tmp/instantclient && \
    unzip -qo '*.zip' -d /opt/oracle
RUN ln -s /opt/oracle/instantclient_12_2 /usr/local/instantclient && \
    ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so && \
    ln -s /usr/local/instantclient/lib* /usr/lib && \
    ln -s /usr/local/instantclient/sqlplus /usr/bin/sqlplus
RUN echo 'export LD_LIBRARY_PATH="/usr/local/instantclient"' >> /root/.bashrc
RUN echo 'instantclient,/usr/local/instantclient' | pecl install oci8
RUN docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/usr/local/instantclient
RUN docker-php-ext-enable oci8 \
    && docker-php-ext-install pdo_oci
RUN rm -rf /tmp/instantclient

# install extensions
RUN apt-get install -qq \
    libsybdb5 \
    openssl \
    unixodbc \
    && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so.5 /usr/lib/libsybdb.so

RUN apt-get install -qq \
    $PHPIZE_DEPS \
    freetds-dev \
    libmagickwand-dev \
    libonig-dev \
    unixodbc-dev

RUN docker-php-ext-configure pdo_odbc --with-pdo-odbc=unixODBC,/usr

RUN docker-php-ext-install -j$(nproc) \
    pdo_odbc \
    pdo

# clean apt
RUN rm -rf /var/lib/apt/lists/*
ENV DEBIAN_FRONTEND=dialog
