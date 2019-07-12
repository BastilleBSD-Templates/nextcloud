Bastille Template to create a Nextcloud Jail

To configure Let's Encrypt you need to issue the following command for the right domain and the 
webroot:

	acme.sh --issue -d <domain> -w <webroot>

Any issues with this refer to the online documentation for Let's Encrypt


Secure your setup

	pw groupmod certs -M acme,www
	chown -R www:certs /usr/local/etc/apache24/ssl
	chmod -R 770 /usr/local/etc/apache24/ssl


Secure mariadb
	/usr/local/bin/mysql_secure_installation

Now Prepare the Database

Login to the database.

	# mysql -u root -p

	Create user nextcloud_admin and add a password.
	]CREATE DATABASE nextcloud;
	CREATE USER 'nextcloud_admin'@'localhost' IDENTIFIED BY '<your password here>';
	GRANT ALL ON nextcloud.* TO 'nextcloud_admin'@'localhost';

	FLUSH PRIVILEGES;

	exit

With the database configured, it can now be restarted.

	# service mysql-server restart


 Configure Web Server
Configure Apache and PHP and confirm they are working properly.

 Apache
Uncomment rewrite_module and ssl_module in /usr/local/etc/apache24/httpd.conf.

Add the location for the certificate we configured earlier.

	SSLCertificateFile /usr/local/etc/apache24/ssl/fullchain.cer
	SSLCertificateKeyFile /usr/local/etc/apache24/ssl/<domain name>.key

Make sure the ssl and rewrite modules are uncommented.

	LoadModule ssl_module libexec/apache24/mod_ssl.so
	LoadModule rewrite_module libexec/apache24/mod_rewrite.so

Make sure the php7 module is uncommented.

	LoadModule php7_module        libexec/apache24/libphp7.so

After libphp7.so line add

	<IfModule php7_module>
	   <FilesMatch "\.(php|phps|php7|phtml)$">
	       SetHandler php7-script
	   </FilesMatch>
	   DirectoryIndex index.php
	</IfModule>

Inside the IFModule mime_module block add:

	AddType application/x-httpd-php-source .phps
	AddType application/x-httpd-php        .php
	Add a PHP handler, in /usr/local/etc/apache24/modules.d/001_mod_php.conf.

	<FilesMatch "\.php$">
	    SetHandler application/x-httpd-php
	</FilesMatch>
	<FilesMatch "\.phps$">
	    SetHandler application/x-httpd-php-source
	</FilesMatch>

Restart apache:

	# service apache24 restart


PHP Configuration

	# cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini && rehash


Here are some standard configuration options for php.ini:

	cgi.fix_pathinfo=0 - Change cgi.fix_pathinfo=1 to cgi.fix_pathinfo=0
	date.timezone = UTC - Set date.timezone to your timezone zone, or UTC.
	post_max_size = 10240M - This is the maximum size of POST data accepted by PHP. Setting 
		it to zero removes the limit.
	upload_max_filesize = 10240M - Maximum allowed size for uploaded files.
	memory_limit = 512M - Adjust memory limit, change this based on your server.


Nextcloud recommends these settings in the php.ini to enable opcache:

	opcache.enable=1
	opcache.enable_cli=1
	opcache.interned_strings_buffer=8
	opcache.max_accelerated_files=10000
	opcache.memory_consumption=128
	opcache.save_comments=1
	opcache.revalidate_freq=1


After that restart apache again with:
	service apache24 restart


Now open the php info in a browser with:
	http://<IP ADDRESS>/info.php


CRON Configuration

Enable the Nextcloud crontab which runs periodic tasks.

	# crontab -u www -e

	*/15 * * * * /usr/local/bin/php -f /usr/local/www/apache24/data/nextcloud/cron.php

Verify it’s scheduled with:

	# crontab -u www -l


Most of the configuration of this template was done with:

	https://ramsdenj.com/2017/06/05/nextcloud-in-a-jail-on-freebsd.html

There are several sections starting at Nextcloud Providers that you should read and
follow to set up the providers and configs you need.


