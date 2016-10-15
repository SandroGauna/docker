FROM debian:jessie
MAINTAINER João Paulo de Souza <vibedigital@gmail.com>
ENV DEBIAN_FRONTEND noninteractive
# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            node-less \
            node-clean-css \
            python-pyinotify \
            python-renderpm \
            python-support \
        && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install brazilian localization dependencies
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            python-pip \
            python-dev \
	    libxml2-dev \
	    libxmlsec1-dev \
	    libjpeg-dev \ 
            build-essential \
	    python-openssl \
	    git

ENV LOCALES_DEF pt_BR.UTF-8 en_US.UTF-8

RUN apt-get update &&  \
    apt-get upgrade -y &&  \
    apt-get -o Dpkg::Options::='--force-confnew' install locales -y

# Uncomment the necessary locales (LOCALES_DEF)

RUN set -- junk $LOCALES_DEF  \
    shift;  \
    for THE_LOCALE in $LOCALES_DEF; do  \
        set --  $(echo $THE_LOCALE  |  awk -F'.' '{ print $1, $2 }');  \
        INPUT_FILE=$1;  \
        CHARMAP_FILE=$2;  \
        localedef  --no-archive -c -i $INPUT_FILE -f $CHARMAP_FILE $THE_LOCALE;  \
    done ; \
    locale-gen

RUN echo "generated locales are:" && locale -a

ENV PYTHON_EGG_CACHE /tmp/.python-eggs
RUN pip install http://labs.libre-entreprise.org/frs/download.php/897/pyxmlsec-0.3.1.tar.gz
RUN git clone https://github.com/Trust-Code/PySPED.git /tmp/pysped
WORKDIR /tmp/pysped
RUN python setup.py install

# Install Odoo
ENV ODOO_VERSION 8.0
ENV ODOO_RELEASE latest
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        #&& echo '6679fad48c761cf8587faa243afcfc17f5a9eb73 odoo.deb' | sha1sum -c - \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

#brazilian localization
RUN git clone https://github.com/oca/l10n-brazil.git --branch 8.0 --depth 1 /mnt/addons/l10n-brazil
RUN git clone https://github.com/oca/account-fiscal-rule.git --branch 8.0 --depth 1 /mnt/addons/account-fiscal-rule
RUN git clone https://github.com/odoo-brazil/odoo-brazil-eletronic-documents.git --branch 8.0 --depth 1 /mnt/addons/odoo-brazil-eletronic-documents
RUN git clone https://github.com/OCA/server-tools --branch 8.0 --depth 1 /mnt/addons/server-tools

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
