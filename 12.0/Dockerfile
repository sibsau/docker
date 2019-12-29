FROM debian:stretch
MAINTAINER Anton Goroshkin <neobht@sibsau.ru>

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
    apt-get update \
    && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    dirmngr \
    fonts-noto-cjk \
    gnupg \
    libssl1.0-dev \
    node-less \
    python3-pip \
    python3-pyldap \
    python3-qrcode \
    python3-renderpm \
    python3-setuptools \
    python3-vobject \
    python3-watchdog \
    xz-utils \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb \
    && echo '7e35a63f9db14f93ec7feeb0fce76b30c08f2057 wkhtmltox.deb' | sha1sum -c - \
    && dpkg --force-depends -i wkhtmltox.deb\
    && apt-get -y install -f --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install rtlcss (on Debian stretch)
RUN set -x;\
    echo "deb http://deb.nodesource.com/node_8.x stretch main" > /etc/apt/sources.list.d/nodesource.list \
    && export GNUPGHOME="$(mktemp -d)" \
    && repokey='9FD3B784BC1C6FC31A8A0A1C1655A0AB68576280' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --armor --export "${repokey}" | apt-key add - \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g rtlcss \
    && rm -rf /var/lib/apt/lists/*

# Install Odoo
ENV ODOO_VERSION 12.0
ARG ODOO_RELEASE=20191229
ARG ODOO_SHA=62589a47f242bad40507611e4793169df40c5c3e

RUN set -x; \
    curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
    && echo "${ODOO_SHA} odoo.deb" | sha1sum -c - \
    && dpkg --force-depends -i odoo.deb \
    && apt-get update \
    && apt-get -y install -f --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* odoo.deb


# Install Postgres to support pg_dump
RUN set -ex; \
    # pub   4096R/ACCC4CF8 2011-10-13 [expires: 2019-07-02]
    #       Key fingerprint = B97B 0AFC AA1A 47F0 44F2  44A0 7FCC 7D46 ACCC 4CF8
    # uid                  PostgreSQL Debian Repository
    key='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8'; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
    gpg --export "$key" > /etc/apt/trusted.gpg.d/postgres.gpg; \
    command -v gpgconf > /dev/null && gpgconf --kill all; \
    rm -rf "$GNUPGHOME"; \
    apt-key list

ENV PG_MAJOR 12
ENV PG_VERSION 12.1-1.pgdg90+1

RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main' $PG_MAJOR > /etc/apt/sources.list.d/pgdg.list

RUN apt-get update \
    && apt-get install -y postgresql-common \
    && sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
    && apt-get install -y \
    postgresql-$PG_MAJOR=$PG_VERSION \
    rlwrap pg-activity \
    && rm -rf /var/lib/apt/lists/*

# Install some Python modules
# RUN set -x; \
#     apt-get update \
#     && apt-get install -y --no-install-recommends \
#         python3-wheel \
#         python3-numpy \
# 	#build-essential python3-dev libldap2-dev libsasl2-dev ldap-utils python-tox lcov valgrind \
#         && rm -rf /var/lib/apt/lists/* \
#     && 

RUN pip3 install \ 
    bs4 PyPDF2 transliterate xlrd xlwt odfpy openpyxl SQLAlchemy \
    config caldav configs phonenumbers \
    cachetools>=2.0.1 cerberus pyquerystring parse-accept-language jsondiff \
    ipdb \
    validate-email==1.3 py3DNS==3.2.0 \
    odoo-client-lib \
    pymorphy2  pymorphy2-dicts-ru
    
RUN set -x; \
    apt-get update \
    && apt-get install -y --no-install-recommends \
    git wget \
    && rm -rf /var/lib/apt/lists/* \
    && pip3 install \
    git+https://github.com/adhoc-dev/aeroolib@master-fix-ods \
    git+https://github.com/aeroo/currency2text.git \
    git+https://github.com/edgewall/genshi@stable/0.7.x

# Copy entrypoint script and Odoo configuration file
RUN pip3 install num2words
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/
RUN chown odoo /etc/odoo/odoo.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
