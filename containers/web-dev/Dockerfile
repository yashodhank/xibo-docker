FROM ubuntu:14.04
MAINTAINER Dan Garner <dan@springsignage.com>

RUN apt-get update && apt-get install -y \
  software-properties-common

RUN DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:chris-lea/zeromq
RUN DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:alexharrington/php-zmq

# Install apache, PHP, and supplimentary programs.
RUN apt-get update && apt-get install -y \
  apache2 \
  libapache2-mod-php5 \
  libapache2-mod-xsendfile \
  mysql-client \
  php5-mysql \
  php5-gd \
  php5-curl \
  php5-cli \
  curl \
  php5-mcrypt \
  php5-zmq \
  ssmtp \
  wget \
  anacron

# Enable apache mods and PHP extensions
RUN a2enmod php5 && a2enmod rewrite && a2enmod env && php5enmod mcrypt && a2enmod xsendfile

# Update the PHP.ini file
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php5/apache2/php.ini
RUN sed -i "s/post_max_size = .*$/post_max_size = 1999M/" /etc/php5/apache2/php.ini
RUN sed -i "s/upload_max_filesize = .*$/upload_max_filesize = 1999M/" /etc/php5/apache2/php.ini
RUN sed -i "s/max_execution_time = .*$/max_execution_time = 300/" /etc/php5/apache2/php.ini
RUN sed -i "s/session.gc_probability = .*$/session.gc_probability = 1/" /etc/php5/apache2/php.ini
RUN sed -i "s/session.gc_divisor = .*$/session.gc_divisor = 100/" /etc/php5/apache2/php.ini

# Disable cron sending emails to root
RUN awk '/PATH=/ { print; print "MAILTO=\"\""; next}1' /etc/crontab > /tmp/crontab && mv /tmp/crontab /etc/crontab
RUN awk '/LOGNAME=root/ { print; print "MAILTO=\"\""; next}1' /etc/anacrontab > /tmp/anacrontab && mv /tmp/anacrontab /etc/anacrontab

# Setup persistent environment variables
ENV CMS_VERSION=1.8.0-rc2 XMR_HOST=xmr CMS_DB_VERSION=129 CMS_SERVER_NAME=localhost
ENV CMS_DATABASE_HOST=mysql CMS_DATABASE_USERNAME=cms CMS_DATABASE_PASSWORD=none CMS_DATABASE_PORT=3306 CMS_DATABASE_NAME=cms
ENV CMS_SERVER=smtp.gmail.com:587 CMS_SMTP_USERNAME=none CMS_SMTP_PASSWORD=none CMS_SMTP_USE_TLS=YES CMS_SMTP_USE_STARTTLS=YES CMS_SMTP_REWRITE_DOMAIN=gmail.com CMS_SMTP_HOSTNAME=none CMS_SMTP_FROM_LINE_OVERRIDE=YES

# Expose port 80
EXPOSE 80

# Pull in the release archive for the environment
RUN mkdir -p /var/www/cms && curl -o cms.tar.gz -SL https://github.com/xibosignage/xibo-cms/releases/download/${CMS_VERSION}/xibo-cms-${CMS_VERSION}.tar.gz \
	&& tar --strip=1 -xzf cms.tar.gz -C /var/www/cms \
	&& rm cms.tar.gz && rm /var/www/cms/web/install/index.php
ADD https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh  /usr/local/bin/wait-for-it.sh
ADD settings.php-template /var/www/cms/web/settings.php
ADD settings.php-template /tmp/settings.php-template
ADD ssmtp.conf /etc/ssmtp/ssmtp.conf
ADD settings-custom.php /tmp/settings-custom.php

# Map a volumes to this folder.
# Our CMS files, library, cache and backups will be in here.
RUN mkdir -p /var/www/cms/library/temp && mkdir -p /var/www/backup && mkdir -p /var/www/cms/cache && mkdir -p /var/www/cms/web/userscripts && chown -R www-data:www-data /var/www/cms

# Update the default apache site with the config we created.
COPY apache-config.conf /etc/apache2/sites-enabled/000-default.conf

# Copy up the various provisioning scripting
COPY entrypoint.sh /entrypoint.sh
COPY httpd-foreground /usr/local/bin/httpd-foreground
RUN chmod +x /entrypoint.sh /usr/local/bin/httpd-foreground /usr/local/bin/wait-for-it.sh

# Create a flag file which the bootstrapping process will delete
# This tells us if it's the first run of a new container
RUN touch /CMS-FLAG

# Add a group for SSMTP
RUN /usr/sbin/groupadd ssmtp

CMD ["/entrypoint.sh"]
