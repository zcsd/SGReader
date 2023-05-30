FROM debian:11-slim

ENV TZ UTC
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && \
	apt-get install --no-install-recommends -y \
	ca-certificates cron \
	apache2 libapache2-mod-php \
	php-curl php-gmp php-intl php-mbstring php-xml php-zip \
	php-sqlite3 php-mysql php-pgsql && \
	rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/www/FreshRSS/ /run/apache2/
WORKDIR /var/www/FreshRSS

COPY . /var/www/FreshRSS
COPY ./Docker/*.Apache.conf /etc/apache2/sites-available/

ARG FRESHRSS_VERSION
ARG SOURCE_BRANCH
ARG SOURCE_COMMIT

RUN a2dismod -f alias autoindex negotiation status && \
	a2enmod deflate expires headers mime remoteip setenvif && \
	a2disconf '*' && \
	a2dissite '*' && \
	a2ensite 'FreshRSS*'

RUN sed -r -i "/^\s*(CustomLog|ErrorLog|Listen) /s/^/#/" /etc/apache2/apache2.conf && \
	sed -r -i "/^\s*Listen /s/^/#/" /etc/apache2/ports.conf && \
	# Disable built-in updates when using Docker, as the full image is supposed to be updated instead.
	sed -r -i "\\#disable_update#s#^.*#\t'disable_update' => true,#" ./config.default.php && \
	touch /var/www/FreshRSS/Docker/env.txt && \
	echo "1,31 * * * * . /var/www/FreshRSS/Docker/env.txt; \
		su www-data -s /bin/sh -c 'php /var/www/FreshRSS/app/actualize_script.php' \
		2>> /proc/1/fd/2 > /tmp/FreshRSS.log" > /etc/crontab.freshrss.default

ENV COPY_LOG_TO_SYSLOG On
ENV COPY_SYSLOG_TO_STDERR On
ENV CRON_MIN ''
ENV FRESHRSS_ENV ''
ENV LISTEN ''

ENTRYPOINT ["./Docker/entrypoint.sh"]

EXPOSE 80
# hadolint ignore=DL3025
CMD ([ -z "$CRON_MIN" ] || cron) && \
	. /etc/apache2/envvars && \
	exec apache2 -D FOREGROUND
