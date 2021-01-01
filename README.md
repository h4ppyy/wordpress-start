# install wordpress in ubuntu16.04

I used tools in windows 10
```
cmder
vagrant
virtualbox
```

### 1. make directory
```
mkdir workspace
cd workspace
mkdir wordpress
cd wordpress
````

### 2. download Vagrantfile
```
curl -LO https://raw.githubusercontent.com/h4ppyy/vagrantfile/master/Vagrantfile_ubuntu16.04
mv Vagrantfile_ubuntu16.04 Vagrantfile
```

### 3. modify ip address
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

MEMORY = 2048
CPU_COUNT = 2

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.synced_folder  ".", "/vagrant", disabled: true

  config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", MEMORY.to_s]
      vb.customize ["modifyvm", :id, "--cpus", CPU_COUNT.to_s]
  end
end
```

### 4 make virtualbox
```
vagrant up
```

### 5. connect virtualbox
```
vagrant ssh
```

### 6. install nginx (default 1.10.3 in ubuntu16.04)
```
cd /tmp
curl -LO https://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
sudo apt-get update
sudo apt install -y nginx
nginx -v
sudo service nginx restart
sudo service nginx status
```

### 7. install mysql (default 5.7 in ubuntu16.04)
```
sudo apt-get install mysql-server
mysql --version
sudo service mysql restart
sudo service mysql status
```

### 8. install php
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.1-fpm php7.1-common php7.1-mbstring php7.1-xmlrpc php7.1-soap php7.1-gd php7.1-xml php7.1-intl php7.1-mysql php7.1-cli php7.1-mcrypt php7.1-zip php7.1-curl
```

### 9. set php7.1-fpm .ini file
```
sudo vi /etc/php/7.1/fpm/php.ini
```

```
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
cgi.fix_pathinfo=0
max_execution_time = 360
date.timezone = Asia/Seoul
```

```
sudo service php7.1-fpm restart
sudo service php7.1-fpm status
```

### 10. set wordpress in mysql

```
sudo mysql -u root -p
0000
```
```
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY '0000';
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY '0000' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```
```
mysql -u wordpressuser -p
0000
```

### 11. download wordpress latest

```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -zxvf latest.tar.gz
sudo mv wordpress /var/www/html/wordpress
sudo chown -R www-data:www-data /var/www/html/wordpress/
sudo chmod -R 755 /var/www/html/wordpress/
```

### 12. set wordpress service in nginx

```
sudo vi /etc/nginx/sites-available/wordpress
```

```
server {
    listen 80;
    listen [::]:80;
    root /var/www/html/wordpress;
    index  index.php index.html index.htm;
    server_name  example.com www.example.com;

    client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$args;        
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
```
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
```
```
sudo rm -rf /etc/nginx/sites-enabled/default
```
```
sudo service nginx restart
```

### 13. configure wordpress

```
sudo mv /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```
```
sudo vi /var/www/html/wordpress/wp-config.php
```
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', '0000' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```