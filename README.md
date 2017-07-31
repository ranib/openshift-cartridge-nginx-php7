# OPENSHIFT Nginx PHP 7 Cartridge
<img src="https://raw.githubusercontent.com/ranib/openshift-cartridge-nginx-php7/master/usr/openshift-redhat.jpg"><br />Welcome to the world of [PHP-FPM](http://php.net/manual/en/book.fpm.php) within [OPENSHIFT](https://www.openshift.com/) by [REDHAT](https://www.redhat.com/en).

## What's inside

* Nginx: 1.13.3
* PHP: 7.1.7
* Latest Composer

## Installation
### Web Console
Click [here](https://openshift.redhat.com/app/console/application_type/custom?unlock=true&application_type%5Bcartridges%5D=http%3A%2F%2Fcartreflect-claytondev.rhcloud.com%2Fgithub%2Franib%2Fopenshift-cartridge-nginx-php7) for installation via web console. <br />
Alternatively, you can use this [cartridge definition](http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx-php7) on application creation page.

### Command Line
```
rhc app create <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx-php7
```
## Updates
You can update the binaries from the cartridge without reinstalling. To check for updates, SSH to your app and run this command:

```
update
```
Make sure to have your backup just in case some things went wrong.

## Configuration

### Nginx
Nginx will automatically include `.openshift/nginx.conf.erb` file.

#### Best nginx configuration for Laravel users
Edit file `.openshift/nginx.conf.erb` to looks like this
```
# Enable Gzip
gzip  on;
gzip_http_version 1.0;
gzip_comp_level 2;
gzip_min_length 1100;
gzip_buffers     4 8k;
gzip_proxied any;
gzip_types
# text/html is always compressed by HttpGzipModule
text/css
text/javascript
text/xml
text/plain
text/x-component
application/javascript
application/json
application/xml
application/rss+xml
font/truetype
font/opentype
application/vnd.ms-fontobject
image/svg+xml;
gzip_static on;
gzip_proxied        expired no-cache no-store private auth;
gzip_disable        "MSIE [1-6]\.";
gzip_vary           on;

client_max_body_size 8M;

server {

    	listen  <%= ENV['OPENSHIFT_PHP_IP'] %>:<%= ENV['OPENSHIFT_PHP_PORT'] %>;
    	root    <%= ENV['OPENSHIFT_REPO_DIR'] %>/public;
	
	location ~* ^.+\.(css|js|jpg|jpeg|gif|png|ico|gz|svg|svgz|ttf|otf|woff|eot|mp4|ogg|ogv|webm)$ {
		access_log off;
		log_not_found off;

		# Some basic cache-control for static files to be sent to the browser
		expires max;
		add_header Pragma public;
		add_header Cache-Control "public, must-revalidate, proxy-revalidate";
	}
	
	location @rewrites {
    	rewrite ^(.*)$ /index.php/$1 last;
    }
	
    location / {
    	index index.php  index.html index.htm;
		try_files $uri $uri/ @rewrites;
    }
	
	# pass the PHP scripts to PHP-FPM
	location ~ ^/index\.php(/|$) {
    	fastcgi_pass unix:<%= ENV['OPENSHIFT_PHP_SOCKET'] %>;
    	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    	fastcgi_param PATH_INFO $fastcgi_script_name;
    	include <%= ENV['OPENSHIFT_PHP_DIR'] %>/usr/conf/fastcgi_params;
	}
	# Add trailing slash to */wp-admin requests.
		rewrite /wp-admin$ $scheme://$host$uri/ permanent;
}
```

### PHP-FPM
PHP-FPM will automatically load `.openshift/php-fpm.ini.erb`, `.openshift/php-fpm.conf.erb`, and `.openshift/extension.ini.erb` files.

## Website
The web root directory is `public/`. Make changes to your website there, then commit and push.

## PECL
To install/uninstall pecl extension just run ssh to your application and run `php-pecl install <name> <ver>`
<br />
Example :
```
php-pecl install apcu 5.1.5
php-pecl install mongodb 1.1.7
php-pecl uninstall apcu
php-pecl uninstall mongodb
```

## Scripts
This cartridge comes with different scripts for easy management of your app inside SSH.

* `version` - Get the version of the cartridge binaries.
* `service` - A psuedo `/usr/bin/service` to start and stop services. Example:
    * `service php-fpm start`
	* `service php-fpm stop`
	* `service php-fpm restart`
	* `service nginx start`
    * `service nginx stop`
	* `service nginx restart`
* `update` - Allows automatic update of the cartridge binaries.
    * `update check` - Check for updates
    * `update install` - Install updates
* `php-pecl` - To install/uninstall pecl extension.
    * `php-pecl install <name> <ver>` - Install pecl extension
    * `php-pecl uninstall <name>` - Uninstall pecl extension

## Build nginx wordpress
### 1. create an app:
#### (1.1) create nginx app with php cartridge (if scalable with -s, otherwise remove -s)
```BASH
$ rhc create-app <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx <-s> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-php
```
or add php to existing nginx app (if already created)
```BASH
$ rhc cartridge add -a <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-php
```	
#### (1.2) add database
for MySQL 5.7.17 use this repo
```BASH
$ rhc cartridge add -a <yourapp> https://raw.githubusercontent.com/ranib/openshift-cartridge-mysql/master/metadata/manifest.yml
```
or MySQL 5.5
```BASH
$ rhc cartridge add mysql-5.5 -a <yourapp>
```	
#### (1.3) add phpmyadmin
```BASH
$ rhc cartridge add phpmyadmin-4 -a <yourapp>
```	
#### (1.4) Only if you want to use Redis 3.2.9 for Object cache
```BASH
$ rhc cartridge add -a <yourapp> http://cartreflect-claytondev.rhcloud.com/reflect?github=ranib/openshift-redis-cart
```
### 2. add Wordpress:

Only if you don't have local copy of <yourapp> created above
- `$ git clone <git-uri yourapp>`
- then change to <yourapp> dir
- `$ cd <yourapp>`
- `$ git remote add upstream https://github.com/ranib/wordpress-example`
-	for scalable (only if app created using -s)
-	`$ git remote add upstream https://github.com/ranib/openshift-scalable-wordpress`
- `$ git pull upstream master`

### 3.	edit wp-config.php for Debugging
The following code, inserted in your wp-config.php file, will log all errors, notices, and warnings to a file called debug.log in the wp-content directory. It will also hide the errors so they do not interrupt page generation.
```BASH
// Enable WP_DEBUG mode
define( 'WP_DEBUG', true );

// Enable Debug logging to the /wp-content/debug.log file
define( 'WP_DEBUG_LOG', true );

// Disable display of errors and warnings 
define( 'WP_DEBUG_DISPLAY', false );
@ini_set( 'display_errors', 0 );

// Use dev versions of core JS and CSS files (only needed if you are modifying these core files)
define( 'SCRIPT_DEBUG', true );
```	
### 4. edit wp-config.php file, change 'true' to 'false'(no SSL) in the line below
```BASH
define('FORCE_SSL_ADMIN', false);

	or add after the line (SSL will be limited to rhcloud domain, will have to get SSL Cert for custom domain)

define(‘FORCE_SSL_LOGIN’, true);
```
### 5. edit wp-config.php file, MUST add
```BASH
/** In Nginx, to resolve "You do not have sufficient permissions to access this page" error after http(s) SSL login */
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
	$_SERVER['HTTPS'] = 'on';
```
### 6. edit wp-config.php file, add (before line /* That's all, stop editing! Happy blogging. */)
```BASH
/** Woocommerce plugin recommends to increase WP Memory Limit to 128MB */
define( 'WP_MEMORY_LIMIT', '128M' );

/** path to fastcgi cache directory for Nginx Helper plugin */
define('RT_WP_NGINX_HELPER_CACHE_PATH', getenv('OPENSHIFT_TMP_DIR').'/nginx-cache');
```
### 7. edit wp-config.php file, look for following line
- `require_once(getenv('OPENSHIFT_REPO_DIR') . '.openshift/openshift.inc');`
- change `'.openshift/openshift.inc'` to `'.openshift/openshift.inc.erb'`

### 8. fix conflicts (if any) in action_hooks/deploy
and change wp root directory from `php` to `public`

run this command to make `action_hooks/build` executable
```BASH
$ git update-index --chmod=+x --add .openshift/action_hooks/*
```
### 9. then commit
- `$ git add -A`
- `$ git commit -am 'install wordpress'`
- `$ git push`