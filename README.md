# Ubuntu Server Setup Guide for Multi-PHP Laravel Projects

This guide walks you through setting up an Ubuntu server to support multiple PHP versions, Nginx, MySQL, Git, Node.js, and Laravel. It also covers deploying Laravel and static sites using Nginx.

---

## Prerequisites

* Ubuntu 22.04 or later
* Sudo privileges

---

## 1. System Update and Basic Tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common curl wget zip unzip git nginx build-essential ca-certificates lsb-release apt-transport-https
```

---

## 2. Install PHP (Multiple Versions)

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP versions
sudo apt install -y php7.4 php7.4-fpm php7.4-mysql php7.4-cli php7.4-curl php7.4-mbstring php7.4-xml php7.4-bcmath php7.4-zip php7.4-soap
sudo apt install -y php8.0 php8.0-fpm php8.0-mysql php8.0-cli php8.0-curl php8.0-mbstring php8.0-xml php8.0-bcmath php8.0-zip php8.0-soap
sudo apt install -y php8.1 php8.1-fpm php8.1-mysql php8.1-cli php8.1-curl php8.1-mbstring php8.1-xml php8.1-bcmath php8.1-zip php8.1-soap
sudo apt install -y php8.2 php8.2-fpm php8.2-mysql php8.2-cli php8.2-curl php8.2-mbstring php8.2-xml php8.2-bcmath php8.2-zip php8.2-soap
sudo apt install -y php8.3 php8.3-fpm php8.3-mysql php8.3-cli php8.3-curl php8.3-mbstring php8.3-xml php8.3-bcmath php8.3-zip php8.3-soap
```

---

## 3. PHP CLI Version Management

```bash
sudo update-alternatives --install /usr/bin/php php /usr/bin/php7.4 74
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.0 80
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.1 81
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.2 82
sudo update-alternatives --install /usr/bin/php php /usr/bin/php8.3 83

sudo update-alternatives --config php
```

---

## 4. Configure Nginx with PHP-FPM per Project

### Example: Project1 (PHP 7.4)

```nginx
server {
    listen 80;
    server_name project1.example.com;
    root /var/www/project1/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

### Example: Project2 (PHP 8.1)

```nginx
server {
    listen 80;
    server_name project2.example.com;
    root /var/www/project2/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

## 5. Install and Configure MySQL

```bash
sudo apt install -y mysql-server
sudo mysql_secure_installation

# Log in and create DB
sudo mysql -u root -p
CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 6. Install Git, Node.js (with n), and Composer

```bash
# Git
sudo apt install -y git

# Node.js with n
curl -fsSL https://raw.githubusercontent.com/tj/n/master/bin/n | bash -
sudo n stable

# Optional: install specific version
# sudo n 18.17.1

# Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

---

## 7. Deploy Laravel Applications

### Example: Laravel Project Structure

```bash
cd /var/www
git clone https://github.com/yourusername/your-laravel-project.git project1
cd project1

composer install
cp .env.example .env
php artisan key:generate

# Set correct permissions
sudo chown -R www-data:www-data /var/www/project1
sudo chmod -R 755 /var/www/project1/storage
sudo chmod -R 755 /var/www/project1/bootstrap/cache
```

### Laravel .env Example

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=strong_password
```

---

## 8. Deploy Static Sites

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static-site;

    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## 9. Add SSL with Let's Encrypt (Certbot)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal test
sudo certbot renew --dry-run
```

---

## 10. Switching PHP Versions (CLI and FPM)

### CLI:

```bash
sudo update-alternatives --config php
```

### Web Server (Nginx):

Change `fastcgi_pass` in the site's Nginx config to the appropriate PHP-FPM socket.

```bash
# Then reload nginx
sudo systemctl reload nginx
```

---

## 11. Restart Services When Needed

```bash
sudo systemctl restart php7.4-fpm
sudo systemctl restart php8.1-fpm
sudo systemctl restart nginx
sudo systemctl restart mysql
```

---

## 12. Firewall Configuration

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

---

## Done 🎉

You now have a fully configured Ubuntu server that can serve multiple Laravel apps with different PHP versions using Nginx, and secured with SSL from Let's Encrypt!
