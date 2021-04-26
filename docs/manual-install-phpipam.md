# {php}IPAM: instalación manual

* requisitos

```bash
sudo apt update
sudo apt -y upgrade

sudo apt install git debconf-utils software-properties-common -y
```

* instala mariadb en la VM de `ipam0`

```bash
sudo apt install mariadb-server mariadb-client -y

# verificamos que mariadb esta corriendo
systemctl status mariadb

sudo debconf-get-selections | grep maria

sudo mysql_secure_installation

new root password -> perico
Remove anonymous users? [Y/n] 
Disallow root login remotely? [Y/n] 
Remove test database and access to it? [Y/n] 
Reload privilege tables now? [Y/n] 


mysql -V
mysql  Ver 15.1 Distrib 10.3.27-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2


sudo mysql -u root -pperico <<EOF

CREATE DATABASE phpipam;
GRANT ALL ON phpipam.* TO phpipam@localhost IDENTIFIED BY 'phpipampassword';
FLUSH PRIVILEGES;

EOF

```

* instalar PHP y sus módulos necesarios

```bash
sudo apt-get install software-properties-common lsb-release apt-transport-https ca-certificates wget -y

if [ "$(lsb_release -is)" == "Ubuntu" ]
then
  sudo add-apt-repository ppa:ondrej/php -y
fi

if [ "$(lsb_release -is)" == "Debian" ]
then
  echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
  wget -O - https://packages.sury.org/php/apt.gpg | sudo tee /etc/apt/trusted.gpg.d/php.gpg > /dev/null
fi

sudo apt update

sudo apt install php-pear php7.4-{fpm,mysql,gmp,curl,intl,mbstring,xmlrpc,gd,xml,imap,memcache,pspell,tidy,json} -y

sudo sed -i "s/^memory_limit = .*/memory_limit = 256M/"  /etc/php/7.4/fpm/php.ini

```


```

* descarga phpipam

```bash
sudo git clone --recursive https://github.com/phpipam/phpipam.git /var/www/phpipam
```

* configura phpipam

```bash
sudo cp /var/www/phpipam/config.dist.php /var/www/phpipam/config.php

export DB_PASS="'phpipampassword'"
sudo sed -i "s/\(\$db\['pass'\][ ]*=[ ]*\).*/\1"${DB_PASS}";/"  /var/www/phpipam/config.php
```

* instala apache2

```bash
sudo apt -y install apache2
sudo a2enmod rewrite

sudo apt -y install libapache2-mod-php7.4 php7.4-curl php7.4-xmlrpc php7.4-intl php7.4-gd

sudo sed -i "s/^memory_limit = .*/memory_limit = 256M/"  /etc/php/7.4/apache2/php.ini

sudo systemctl restart apache2

```

* configura apache2


```bash

sudo tee /etc/apache2/sites-available/phpipam.conf <<'EOF'

<VirtualHost *:80>
    ServerAdmin sysadmin@infra.ballardini.com.ar
    DocumentRoot "/var/www/phpipam"
    ServerName ipam.infra.ballardini.com.ar
    ServerAlias www.ipam.infra.ballardini.com.ar
    <Directory "/var/www/phpipam">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/apache2/phpipam-error_log"
    CustomLog "/var/log/apache2/phpipam-access_log" combined
</VirtualHost>
EOF

sudo  a2ensite phpipam
sudo systemctl restart apache2

```

* crea las tablas en la base de datos

```bash
sudo mysql -u root < /vagrant/provision/files/ipam0/phpipam.sql

```

## Web UI config - settings

* credenciales: `admin / ipamadmin`

* cambio contraseña al inicio: `peperuch0`

* http://ipam.infra.ballardini.com.ar/administration/settings/

* Site settings
  * Site title: phpipam IP address management
  * Site domain: infra.ballardini.com.ar
  * Site URL: http://ipam.infra.ballardini.com.ar/
  * Prettify links: Yes	
  * Default language: English
  * Default theme: white

* Admin settings
  * Admin name: Sysadmin
  * Admin mail: sysadmin@infra.ballardini.com.ar

* Feature settings
  * API: ON
  * Enable PowerDNS: ON

* Display settings
  * Hide donation button: OFF

## Web UI config - API

* http://ipam.infra.ballardini.com.ar/administration/api/

* Create API key
  * App id -> PowerDNS
  * App code: qXG-EXDYC0hpvhUArTepRea1L-xFxoCv	
  * App permissions: Read / Write / Admin
  * App security: SSL with User Token
  * Transaction lock: No
  * Lock timeout: 
  * Nest custom fields: No
  * Show links: No
  * Description: -> Interacts with PowerDNS blind server


## Web UI config - PowerDNS

* Database settings
  * Host -> 192.168.33.110
  * Database -> pdns
  * Username -> powerdns
  * Password -> P0w3rDn5
  * Port: 3306
  * Autoserial: OFF

* http://ipam.infra.ballardini.com.ar/administration/powerDNS/defaults/ FIXME set default values

## CLI UI Config - settings

```bash

POWERDNS_CONFIG=$( jq -n \
   --arg host 192.168.33.110 \
   --arg name pdns \
   --arg username powerdns \
   --arg  password P0w3rDn5 \
   --arg port 3306  \
'{
  "host":$host,
  "name":$name,
  "username":$username,
  "password":$password,
  "port":$port,
  "autoserial": "No",
  "ns": null,
  "hostmaster": null,
  "def_ptr_domain": null,
  "refresh": null,
  "retry": null,
  "expire": null,
  "nxdomain_ttl": null,
  "ttl": null
}')

echo $POWERDNS_CONFIG

sudo mysql --table --user root  --database=phpipam << 'EOF'
UPDATE settings SET siteTitle  = 'IP address management infra.ballardini.com.ar' WHERE id = 1;
UPDATE settings SET siteDomain = 'ballardini.com.ar' WHERE id = 1;
UPDATE settings SET siteURL    = 'ipam.infra.ballardini.com.ar' WHERE id = 1;

UPDATE settings SET prettyLinks = 'Yes' WHERE id = 1;
UPDATE settings SET theme       = 'white' WHERE id = 1;
UPDATE settings SET donate      = 1 WHERE id = 1;

UPDATE settings SET siteAdminName = 'sysadmin' WHERE id = 1;
UPDATE settings SET siteAdminMail = 'sysadmin@infra.ballardini.com.ar' WHERE id = 1;

UPDATE settings SET api = 1 WHERE id = 1;
UPDATE settings SET enablePowerDNS = 1 WHERE id = 1;
EOF


sudo mysql --table --user root  --database=phpipam << EOF
UPDATE settings SET powerDNS = '${POWERDNS_CONFIG}' WHERE id = 1;
EOF


sudo mysql --table --user root  --database=phpipam << 'EOF'
select siteTitle,siteDomain,siteURL  from settings;
select prettyLinks,theme from settings;
select siteAdminName,siteAdminMail from settings;
select api,enablePowerDNS,donate from settings;
select powerDNS from settings;
EOF

```

## CLI UI Config - API

* crea un token para la app PowerDNS

```bash
sudo mysql --table --user root  --database=phpipam << 'EOF'
INSERT INTO api (app_id,
                 app_code,
                 app_permissions,
                 app_comment,
                 app_security,
                 app_lock,
                 app_lock_wait,
                 app_nest_custom_fields,
                 app_show_links)
VALUES ('PowerDNS',
        'qXG-EXDYC0hpvhUArTepRea1L-xFxoCv',
        3,
        'Interacts with PowerDNS blind server',
        'ssl_token',
        0,0,0,0);


EOF
```

# Referencias

* https://computingforgeeks.com/install-and-configure-phpipam-on-ubuntu-debian-linux/

