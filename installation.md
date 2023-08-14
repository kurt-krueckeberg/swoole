# Installation of Open Swoole

## Suggested PHP extensions to first have installed:

- curl
- json
- bcmath
- mbstring
- opcache
- xml
- zip

## Installing Open Swoole with Pecl

As stated in Open Swoole [prequisties](https://openswoole.com/docs/get-started/prerequisites). These Ubuntu or Debian apt packages must first be installed:

- openssl
- libssl-dev
- curl
- libcurl4-openssl-dev
- libpcre3-dev
- build-essential

> [!WARNING]
> As of August, 2023, using g++ 13 results in compile errors that terminate the installation. g++ version 12 does work.

[How to Install Open Swoole](https://openswoole.com/docs/get-started/installation) explains you can supply predefined responses to the pecl installation
prompts. Below are my personal installation choices (of "yes" to everything except the native MySQL driver):

```bash
sudo pecl install -D 'enable-sockets="yes" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="no" enable-hook-curl="yes" enable-cares="yes" with-postgres="no"' openswoole
```

The build of the `openswoole.so` library concludes with:

```
Build process completed successfully
Installing '/usr/lib/php/20210902/openswoole.so'
Installing '/usr/include/php/20210902/ext/openswoole/config.h'
Installing '/usr/include/php/20210902/ext/openswoole/php_openswoole.h'
install ok: channel://pecl.php.net/openswoole-22.0.0
configuration option "php_ini" is not set to php.ini location
You should add "extension=openswoole.so" to php.ini
```

Therefore, you must manually add "extension=openswoole.so" to php.ini. I chose to do it this way:

First, since the curl extension must load before swoole, I did this:

1. Created `openswoole.ini` in `/etc/php/8.1/mods-available/openswoole.ini` with this content

```ini
; priority=25.
extension=openswoole.so
```

2. Create a symbolic links in `/etc/php/8.1/cli/conf.d`  named `25-swoole.ini` that refers to
`/etc/php/8.1/mods-available/swoole.ini`:

```bash
cd /etc/php/8.1/cli/conf.d

sudo ln -s /etc/php/8.1/mods-available/openswoole.ini 25-openswoole.ini
```

The '25` ensures it will load after the curl extension, whose symlink is `20-curl.ini`.

> [!NOTE] As mentioned in the Swoole overview, you don't need to enable to php8.X-fpm in order to use the swoole extensions. You DON'T
> need to create a similiar `/etc/php/8.1/fpm/conf.d/25-openswoole.ini` and then restart Apache or `php8.1-fpm` (as of this writing) for Nginx!

To check that `openswoole` was installed:

```
$ php -m | grep openswoole
```

Lastly, enable use of Open Swoole for your project:

```bash
composer require openswoole/core:22.1.2
```
