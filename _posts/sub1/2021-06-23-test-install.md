---
title: "Welcome to Jekyll!"
date: 2017-10-20 08:26:28 -0400
categories: jekyll update
---
## Environment

- PHP >= 7.3
- BCMath PHP Extension
- Ctype PHP Extension
- Fileinfo PHP extension
- JSON PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PDO PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
- 10.4.18-MariaDB
- composer 1.10.x

## Install

- composer

```linux
$ cd ~
$ curl -sS https://getcomposer.org/installer -o composer-setup.php
$ HASH=`curl -sS https://composer.github.io/installer.sig`
$ echo $HASH
$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
$ composer self-update 1.10.12
$ composer --version 
```
- framework

```linux
$ cd ~
$ composer install
```

## other

```linux
$ cd appup
$ cp .env.example .env
$ php artisan key:generate
$ sudo chmod 777 -R storage
$ php artisan storage:link
```

## test

```linux
$ wget localhost
$ cat index.html
```
