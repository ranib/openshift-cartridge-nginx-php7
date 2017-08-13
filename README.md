# OPENSHIFT Nginx PHP 7 Cartridge
<img src="https://raw.githubusercontent.com/ranib/openshift-cartridge-nginx-php7/master/usr/openshift-redhat.jpg"><br />Welcome to the world of [PHP-FPM](http://php.net/manual/en/book.fpm.php) within [OPENSHIFT](https://www.openshift.com/) by [REDHAT](https://www.redhat.com/en).

#OPENSHIFT Nginx PHP 7 Cartridge
<img src="https://raw.githubusercontent.com/ranib/openshift-cartridge-nginx-php7/master/usr/openshift-redhat.jpg"><br />Welcome to the world of [PHP-FPM](http://php.net/manual/en/book.fpm.php) within [OPENSHIFT](https://www.openshift.com/) by [REDHAT](https://www.redhat.com/en).

## What's inside
* Nginx: 1.13.3 (compiled with ngx_http_redis_module)
* PHP: 7.1.8
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
php-pecl install redis 3.1.3
php-pecl uninstall apcu
php-pecl uninstall mongodb
php-pecl uninstall redis
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

## Compiling a new version

* To compile a new version create a new openshift application (or use existing one).
	* `$ rhc create-app <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx-php7`
	* Now clone <yourapp> and create `nginx` folder. 
	* Now copy openshift-cartridge-nginx-php7/usr/`compile` folder and paste into `nginx` folder
	* Now set the versions you need to compile in the nginx/compile/`versions` file. 
	* Edit `build` if required and make it executable `git update-index --chmod=+x --add nginx/compile/*` 
	* Commit and push <yourapp>

* SSH into your app `rhc ssh <yourapp>` and go to the compile folder `cd ${OPENSHIFT_REPO_DIR}/nginx/compile` and start compiling by running the following command: `./all`

* Once compiling is done download `nginx-{version}.tar.gz` from <yourapp>`/public` directory.
	* Extract `nginx-{version}.tar.gz` and place only `nginx` file into openshift-cartridge-nginx-php7/usr/`sbin` folder.
	* place (if created) `nginx-module.so` file from `module` folder into openshift-cartridge-nginx-php7/usr/`ext` folder.
	* If modified compile/`build` before compiling, copy and place it into openshift-cartridge-nginx-php7/usr/`compile` folder.
	* Delete the `nginx-{version}` locally and from <yourapp>`/public` directory, you do not need any more.
	* Change permissions to make `nginx` executable `git update-index --chmod=+x --add usr/sbin/nginx`
	* Change permissions to make `nginx-module.so` executable `git update-index --chmod=+x --add usr/ext/nginx-module.so`
	* Modify the value of NGINX_VERSION in /metadata/`manifest.yml` (if required) in /bin/`setup`
	* Commit and push
	
## Build nginx wordpress
### 1. create an app:
#### (1.1) create nginx app with php cartridge (if scalable, require more than 1 gear, use -s, otherwise remove -s)
```BASH
$ rhc create-app <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx-php7 <-s>
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
### 2. add Wordpress

Only if you don't have local copy of <yourapp> created above
- `$ git clone <git-uri yourapp>`
- then change to <yourapp> dir
- `$ cd <yourapp>`
- `$ git remote add upstream https://github.com/ranib/wordpress-example`
-	 for scalable (only if app created using -s)
-   `$ git remote add upstream https://github.com/ranib/openshift-scalable-wordpress`
- `$ git pull upstream master`

### 3. edit wp-config.php file
```BASH
/** change 'true' to 'false'(no SSL) */
define('FORCE_SSL_ADMIN', false);

/** or leave it as is (SSL will be limited to rhcloud domain, will have to get SSL Cert for custom domain) */
define(‘FORCE_SSL_LOGIN’, true);

/** In Nginx, to resolve "You do not have sufficient permissions to access this page" error after http(s) SSL login */
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
	$_SERVER['HTTPS'] = 'on';

/** Woocommerce plugin recommends to increase WP Memory Limit to 128MB */
define( 'WP_MEMORY_LIMIT', '128M' );

/** path to fastcgi cache directory for Nginx Helper plugin */
define('RT_WP_NGINX_HELPER_CACHE_PATH', getenv('OPENSHIFT_TMP_DIR').'/nginx-cache');

/** OPTIONAL - for Wordpress Debugging ONLY if necessary
The following code, inserted in your wp-config.php file, will log all errors, notices, and warnings to a file called debug.log in the wp-content directory. 
It will also hide the errors so they do not interrupt page generation. */

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
### 4. fix conflicts (if any) in action_hooks/deploy
and change wp root directory from `php` to `public`

you can also use `composer` to install `wordpress themes & plugins`

to make `action_hooks/build` executable run this command
```BASH
$ git update-index --chmod=+x --add .openshift/action_hooks/*
```
then commit
- `$ git add -A`
- `$ git commit -am 'install wordpress'`
- `$ git push`

### 5. After installing Wordpress it is time to tweak Nginx for best performance
* Edit Nginx configuration file `.openshift/nginx.conf.erb` with necessary Wordpress rules (since .htacess file used in Apache does not work in Nginx)
