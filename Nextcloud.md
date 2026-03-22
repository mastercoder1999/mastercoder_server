# Cloud Personel
- Nextcloud
- MariaDB
- PostgreSQL

## MariaDB
### Install
```
sudo apt install mariadb-server -y

sudo systemctl enable mariadb
sudo systemctl start mariadb
```

### Secure DB
```
sudo mysql_secure_installation
```

- Enter (leave empty )
- n
- n
- y
- y
- y
- y

### Créer DB Nextcloud
```
sudo mysql -u root -p
```
```
CREATE DATABASE nextcloud;
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'TON_MDP_ICI';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Nextcloud
### Préparation
```
sudo mkdir -p /srv/nextcloud/data
```

### Install
```
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
sudo apt install unzip -y
unzip latest.zip

sudo mv nextcloud /var/www/
```

### Permissions
```
sudo chown -R www-data:www-data /var/www/nextcloud
sudo chown -R www-data:www-data /srv/nextcloud
sudo chmod -R 750 /var/www/nextcloud
sudo chmod -R 750 /srv/nextcloud
```

## PHP
```
sudo apt install php php-fpm php-mysql php-xml php-gd php-curl php-zip php-mbstring php-intl php-bcmath php-gmp -y
``` 

## NGINX
```
sudo nano /etc/nginx/sites-available/nextcloud
```
```
server {
    server_name cloud.maksexperience.xyz;

    root /var/www/nextcloud;
    index index.php index.html;

    client_max_body_size 512M;

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }

    listen 80;
}
```

```
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx
```