FROM alpine:latest
MAINTAINER Anton Goroshkin <neobht@sibsau.ru>

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apk update \
        && \ 
        apk add py-pip \
                curl \
                nodejs \
		bash \
		linux-headers postgresql-dev gcc python-dev musl-dev libxslt-dev openldap-dev \
	&& rm -rf /var/cache/apk/* \
        && npm install -g less

# Install Odoo
ENV ODOO_VERSION 9.0
ENV ODOO_RELEASE 20161231
RUN set -x; \
        curl -o odoo.tar.gz -SL https://nightly.odoo.com/${ODOO_VERSION}/nightly/src/odoo_${ODOO_VERSION}c.${ODOO_RELEASE}.tar.gz \
        # && echo 'e7f7cabcb760c90543b3a113a0f5676da5f9c314 odoo.rpm' | sha1sum -c - \
        && tar -xvf odoo.tar.gz \
        && /bin/rm odoo.tar.gz

RUN set -x; \
	pip install --no-cache-dir --upgrade -r odoo-${ODOO_VERSION}c-${ODOO_RELEASE}/requirements.txt

# Install some Python modules
RUN set -x; \
	pip install --no-cache-dir PyPDF2 transliterate xlrd odfpy

RUN set -x; \
	cd odoo-${ODOO_VERSION}c-${ODOO_RELEASE} && python setup.py install \
	&& rm -rf odoo-${ODOO_VERSION}c-${ODOO_RELEASE}
# Install Aeroo Reports support
# ADD ./python-aeroolib_1.0_all.deb /
# RUN set -x; \
#         apt-get update \
#         && dpkg --force-depends -i  python-aeroolib_1.0_all.deb \ 
#         && apt-get -y install -f --no-install-recommends \
#         && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
#         && rm -rf /var/lib/apt/lists/* python-aeroolib_1.0_all.deb


# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN adduser -H -S odoo
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
