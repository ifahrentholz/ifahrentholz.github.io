---
title:  "Install Owncloud on Raspberry"
date:   2016-08-10 15:04:23
categories: [raspberry, owncloud]
tags: [raspberry, owncloud]
---

In the following steps I'll document how the owncloud setup works.


Configure your Pi

```bash
sudo raspi-config
```

1. Expand Filesystem
2. Change user password
3. Change hostname
4. Set boot to CLI
5. Set GPU Memory to 16
6. Update your localisation settings

Update your system

```
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
```

Add the www-data user to the www-data group:

```
sudo usermod -a -G www-data www-data
```

Install the required packages

```
sudo apt-get install nginx openssl ssl-cert php5-cli php5-sqlite php5-gd php5-common php5-cgi sqlite3 php-pear php-apc curl libapr1 libtool curl libcurl4-openssl-dev php-xml-parser php5 php5-dev php5-curl php5-gd php5-fpm memcached php5-memcache varnish
```

Create the SSL certificat

```
sudo openssl req $@ -new -x509 -days 730 -nodes -out /etc/nginx/cert.pem -keyout /etc/nginx/cert.key
``

Protect the certificat

```
sudo chmod 600 /etc/nginx/cert.pem
sudo chmod 600 /etc/nginx/cert.key
```

Update the nginx configuration

```
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default_old
sudo nano /etc/nginx/sites-available/default
```

Paste the following configuration. Make shure you substitue **IPAdress** with your Raspberry Pi IPAdress

```
upstream php-handler {
    server 127.0.0.1:9000;
    #server unix:/var/run/php5-fpm.sock;
}

server {
    listen 80;
    server_name IPaddress;
    return 301 https://$server_name$request_uri; # enforce https
}

server {
    listen 443 ssl;
    server_name IPaddress;
    ssl_certificate /etc/nginx/cert.pem;
    ssl_certificate_key /etc/nginx/cert.key;
    # Path to the root of your installation
    root /var/www/owncloud;
    client_max_body_size 1000M; # set max upload size
    fastcgi_buffers 64 4K;
    rewrite ^/caldav(.*)$ /remote.php/caldav$1 redirect;
    rewrite ^/carddav(.*)$ /remote.php/carddav$1 redirect;
    rewrite ^/webdav(.*)$ /remote.php/webdav$1 redirect;
    index index.php;
    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
    location ~ ^/(?:\.htaccess|data|config|db_structure\.xml|README) {
        deny all;
    }
    location / {
        # The following 2 rules are only needed with webfinger
        rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
        rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;
        rewrite ^/.well-known/carddav /remote.php/carddav/ redirect;
        rewrite ^/.well-known/caldav /remote.php/caldav/ redirect;
        rewrite ^(/core/doc/[^\/]+/)$ $1/index.html;
        try_files $uri $uri/ index.php;
    }
    location ~ \.php(?:$|/) {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        fastcgi_pass php-handler;
    }
    # Optional: set long EXPIRES header on static assets
    location ~* \.(?:jpg|jpeg|gif|bmp|ico|png|css|js|swf)$ {
         expires 30d;
         # Optional: Don't log access to assets
         access_log off;
    }
}
```

Adjust the php.ini

```
sudo nano /etc/php5/fpm/php.ini
```

Use **Ctrl+W** to search for the lines below

```
upload_max_filesize = 2000M
post_max_size = 2000M
```

Adjust the www.conf

```
sudo nano /etc/php5/fpm/pool.d/www.conf
```

Change listen = /var/run/php5-fpm.sock
to
listen = 127.0.0.1:9000

Adjust the Swapsize

```
sudo nano /etc/dphys-swapfile
```

CONF_SWAPSIZE=100
to
CONF_SWAPSIZE=512

Restart your pi

```
sudo reboot
```

### Install Owncloud

```
sudo mkdir -p /var/www/owncloud
sudo wget https://download.owncloud.org/community/owncloud-9.1.0.tar.bz2
sudo tar xvf owncloud-9.1.0.tar.bz2
sudo mv owncloud/ /var/www/
sudo chown -R www-data:www-data /var/www
rm -rf owncloud owncloud-9.1.0.tar.bz2
```

### Mount your drive

Install the following packages

```
sudo apt-get install ntfs-3g exfat-utils
```

Create a directory we can mount to

```
sudo mkdir /media/ownclouddrive
```

Get your gid, uid and UUID

- gid

```
id -g www-data

```

- uid

```
id -u www-data
```

- UUID

```
ls -l /dev/disk/by-uuid
```

Adjust fstab

```
sudo nano /etc/fstab
```

At the bottom paste the following

```
UUID=F6941E59941E1D25 /media/ownclouddrive auto nofail,uid=33,gid=33,umask=0027,dmask=0027,noatime 0 0
```

Test if everything is ok

```
sudo mount -a
```

If you don't get any errors reboot

```
sudo reboot
```

Now you can open your webbrowser and setup owncloud.
Please make sure you configure your Data folder below **Storage & database**