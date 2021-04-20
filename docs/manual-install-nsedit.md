# nsedit: Instalación manual

## 1. requisitos previos

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release git -y
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y

```


## 2. nginx

```bash
echo "deb http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list

curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt update
sudo apt install nginx -y

sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```

## 3. PHP

```bash
sudo apt-get install software-properties-common lsb-release apt-transport-https ca-certificates wget -y
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
wget -O - https://packages.sury.org/php/apt.gpg | sudo tee /etc/apt/trusted.gpg.d/php.gpg > /dev/null

# sudo add-apt-repository ppa:ondrej/php -y # ubuntu
# sudo add-apt-repository ppa:ondrej/nginx-mainline -y # ubuntu

sudo apt update

sudo apt install php7.4-fpm php7.4-common php7.4-mysql php7.4-gmp php7.4-curl php7.4-intl php7.4-mbstring php7.4-xmlrpc php7.4-gd php7.4-xml php7.4-cli php7.4-zip php7.4-soap php7.4-imap php7.4-sqlite3 -y

sudo sed -i "s/^memory_limit = .*/memory_limit = 256M/"  /etc/php/7.4/fpm/php.ini

```

## 4. configura nginx

```bash
sudo usermod -a -G www-data nginx
sudo chown -R www-data /usr/share/nginx/html


sudo tee /etc/nginx/conf.d/default.conf  <<'EOF'

server {
    listen       80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.php index.html index.htm;

   location / {
    if ($request_uri ~ ^/(.*)\.html$) {
        return 302 /$1;
        }
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
        location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass   unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}
EOF

sudo tee /usr/share/nginx/html/phpinfo.php <<EOF
<?php
phpinfo();
?>
EOF

sudo chown www-data:root /usr/share/nginx/html/phpinfo.php

sudo service nginx restart
```

* http://192.168.33.10/phpinfo.php debe mostrar la info de PHP instalado

## 5. instala nsedit

```bash
cd /usr/share/nginx/html/
sudo git clone https://github.com/tuxis-ie/nsedit.git

cd nsedit/
sudo git checkout tags/v1.0
```

* configurar nsedit:

```bash
sudo mkdir /usr/local/etc/nsedit/
sudo chown -R www-data /usr/local/etc/nsedit/

sudo cp includes/config.inc.php-dist includes/config.inc.php
```


* edit `includes/config.inc.php` para que coincida con lo configurado en `site.yml` para PowerDNS

```php
$apipass = 'powerdns';       # The PowerDNS API-key
$apiip   = '192.168.33.10';  # The IP of the PowerDNS API
$apiport = '8001';       # The port of the PowerDNS API

$authdb  = "/usr/local/etc/nsedit/pdns.users.sqlite3";
```

* http://192.168.33.10/nsedit/


# Referencias

* https://github.com/tuxis-ie/nsedit
* https://hostup.org/blog/how-to-install-nginx-and-php-7-4-on-debian-10/
