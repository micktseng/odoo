FROM debian:stretch
MAINTAINER Mick Tseng <suncombo@gmail.com>

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            git \
            node-less \
            python3-pip \
            python3-setuptools \
            python3-renderpm \
            python3-watchdog \
            python3-dev \
            libsasl2-dev \
            libldap2-dev \
            libssl-dev \
            xz-utils \
            build-essential \
            # For wkhtmltox
            libfontconfig1 \
            libxrender1 \
            libxext6 \ 
            # For database management
            postgresql-client-9.6 \
        && curl -o wkhtmltox.tar.xz -SL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
        && echo '3f923f425d345940089e44c1466f6408b9619562 wkhtmltox.tar.xz' | sha1sum -c - \
        && tar xvf wkhtmltox.tar.xz \
        && cp wkhtmltox/lib/* /usr/local/lib/ \
        && cp wkhtmltox/bin/* /usr/local/bin/ \
        && cp -r wkhtmltox/share/man/man1 /usr/local/share/man/ \
        && rm -rf /var/lib/apt/lists/*

# Add odoo user (apply the same in the host machine for compatibility)
RUN addgroup --gid=300 odoo && adduser --system --uid=300 --gid=300 --home /odoo --shell /bin/bash odoo

USER odoo

# Create directories
RUN bin/bash -c "mkdir /odoo/{addons,data,etc,log}"
COPY ./odoo.conf /odoo/etc/

# Get Odoo code
WORKDIR /odoo
RUN set -x; \
  git clone --progress --depth 1 https://github.com/odoo/odoo.git -b 11.0 source \
  && rm -rf odoo/.git

USER 0

# Copy entrypoint script and Odoo configuration file
RUN python3 -m pip install --upgrade pip \
  && pip3 install num2words phonenumbers \
  && pip3 install -r /odoo/source/requirements.txt

COPY ./entrypoint.sh /

VOLUME ["/odoo/addons", "/odoo/data", "/odoo/etc"]

# Expose Odoo services
EXPOSE 8069 8071        

# Set the default config file
ENV ODOO_RC /odoo/etc/odoo.conf

USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
