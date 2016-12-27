FROM fotengauer/altlinux-p8:latest
MAINTAINER Anton Goroshkin <neobht@sibsau.ru>

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && \ 
        apt-get install -y  \
                python-module-pyparsing \
                python-module-dateutils \
                python-module-decorator \
                python-module-docutils \
                python-module-feedparser \
                python-module-imaging \
                python-module-jinja2 \
                python-module-ldap \
                python-module-lxml \
                python-module-mako \
                python-module-mock \
                python-module-openid \
                python-module-passlib \
                python-module-psutil \
                python-module-psycopg2 \
                python-module-Reportlab \
                python-module-requests \
                python-module-vobject \
                python-module-werkzeug \
                python-module-xlwt \
                python-module-odfpy \
                python-module-xlrd \
                python-module-yaml \
                python-module-pytz \
                python-module-pychart \
                python-module-pypdf \
                python-module-vatnumber \
                python-module-pydot \
                python-module-suds \
                python-module-gevent \
                python-module-pip \
                curl \
        # && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        # && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        # && dpkg --force-depends -i wkhtmltox.deb \
        # && apt-get -y install -f --no-install-recommends \
        # && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        # && rm -rf /var/lib/apt/lists/* wkhtmltox.deb \
        && /bin/rm /var/lib/apt/lists/* /var/cache/apt/*.bin /var/lib/rpm/* \
        && apt-get clean \
        && pip install psycogreen==1.0

# Install Odoo
ENV ODOO_VERSION 9.0
ENV ODOO_RELEASE 20161218
RUN set -x; \
        curl -o odoo.rpm -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/rpm/odoo_${ODOO_VERSION}c.${ODOO_RELEASE}.noarch.rpm \
        # && echo 'e7f7cabcb760c90543b3a113a0f5676da5f9c314 odoo.rpm' | sha1sum -c - \
        && rpm -i --nodeps odoo.rpm \
        && /bin/rm odoo.rpm

# Install Aeroo Reports support
# ADD ./python-aeroolib_1.0_all.deb /
# RUN set -x; \
#         apt-get update \
#         && dpkg --force-depends -i  python-aeroolib_1.0_all.deb \ 
#         && apt-get -y install -f --no-install-recommends \
#         && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
#         && rm -rf /var/lib/apt/lists/* python-aeroolib_1.0_all.deb

# # Install some Python modules
# RUN set -x; \
#     apt-get update \
#     && apt-get install -y --no-install-recommends \
#         python-pip \
#         && rm -rf /var/lib/apt/lists/* \
#     && pip install \ 
#         bs4 PyPDF2 transliterate xlrd odfpy

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["openerp-server"]
