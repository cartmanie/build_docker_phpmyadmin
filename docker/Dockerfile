

FROM debian:bookworm-slim as builder

RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
     unzip \
     ca-certificates \
     curl \
	 php-fpm=2:8.2+93 \
	 php-cgi=2:8.2+93  \
	 lighttpd=1.4.69-1  \
	 php8.2-mysql=8.2.20-1~deb12u1 \
     php-mbstring

ENV VERSION 5.2.1
ENV URL https://files.phpmyadmin.net/phpMyAdmin/${VERSION}/phpMyAdmin-5.2.1-all-languages.zip

RUN set -ex; \
    \
curl -fsSL -o phpMyAdmin-5.2.1-all-languages.zip $URL; \
unzip phpMyAdmin-5.2.1-all-languages.zip -d /var/www/html_lighttpd && \
mv /var/www/html_lighttpd/phpMyAdmin-5.2.1-all-languages/* /var/www/html_lighttpd/ && \
rm -rf /var/www/html_lighttpd/phpMyAdmin-5.2.1-all-languages


RUN set -ex; \
    \
grep -q -F "'configFile' => ROOT_PATH . 'config.inc.php'," /var/www/html_lighttpd/libraries/vendor_config.php; \
sed -i "s@'configFile' => .*@'configFile' => '/etc/phpmyadmin/config.inc.php',@" /var/www/html_lighttpd/libraries/vendor_config.php; \
grep -q -F "'configFile' => '/etc/phpmyadmin/config.inc.php'," /var/www/html_lighttpd/libraries/vendor_config.php; \
php -l /var/www/html_lighttpd/libraries/vendor_config.php; \
    chown -R www-data:www-data -R /var/www/html_lighttpd/; 

RUN set -ex; \
    \
    sed -i "s@server.document-root.*=.*@server.document-root        = \"/var/www/html_lighttpd\"@g" /etc/lighttpd/lighttpd.conf; \
    sed -i "s@server.port.*=.*@server.port                 = 18000@g" /etc/lighttpd/lighttpd.conf; \
    grep -q -F 'server.port                 = 18000' /etc/lighttpd/lighttpd.conf; \
    grep -q -F 'server.document-root        = "/var/www/html_lighttpd"' /etc/lighttpd/lighttpd.conf; \
    lighty-enable-mod fastcgi && lighty-enable-mod fastcgi-php;




# Copy configuration
COPY config.inc.php /etc/phpmyadmin/config.inc.php
RUN chown www-data:www-data -R /etc/phpmyadmin/

CMD ["sh", "-c", "service php8.2-fpm start && lighttpd -D -f /etc/lighttpd/lighttpd.conf"]