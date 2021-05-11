# PHP 7.2

## Installing PHP 7.2
To install PHP and all required PHP extensions run the following commands:
```sh
yum install epel-release yum-utils
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpmsudo yum-config-manager --enable remi-php72
yum install php-cli php-fpm php-mysql php-json php-opcache php-mbstring php-xml php-gd php-curl
```

open the `/etc/php-fpm.d/www.conf` file edit the lines:
```Nginx
user = nginx
...
group = nginx
...
listen = /run/php-fpm/www.sock
...
listen.owner = nginx
listen.group = nginx
```

Make sure the `/var/lib/php` directory has the correct ownership using the following chown command:
```sh
sudo chown -R root:nginx /var/lib/php
```

Once you made the changes, enable and start the PHP FPM service:
```sh
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

### Hiding your PHP version
Then open the file using your favorite editor with super user privileges like so:
```sh
sudo vi /etc/php.ini
```

Locate the keyword `expose_php` and set its value to `Off`:
```ini
expose_php = off
```

Save the file and exit.
You can always find php directory location using php* and grep command:
```sh
php -i | grep "Loaded Configuration File"
```

Add the following line to custom.ini as per your setup:
```sh
echo 'expose_php = off' >> /etc/php.d/custom.ini
```

Restart/reload PHP:
```sh
systemctl restart php-fpm.service
```

Use the curl command to check headers:
```sh
curl -IL https://server.com/
```

## Error 413 wordpress
Check and/or increase the following in `php.ini`:
```ini
upload_max_filesize = 32M
post_max_size = 32M
```

Optionally increase:
```ini
max_execution_time =300
max_input_time=300
memory_limit =128M
```