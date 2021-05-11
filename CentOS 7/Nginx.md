# Nginx
## Install
```sh
yum install nginx
systemctl enable nginx
systemctl start nginx
```

## Hide Nginx Server Version
Edit `/etc/nginx/nginx.conf` and add the following line to http context:
```Nginx
server_tokens off;
```

After adding above line, save the file and restart Nginx server to take new changes into effect.
```sh
systemctl restart nginx
```

## Add Virtual domain for wordpress
Create new file `/etc/nginx/conf.d/example.com.conf` and add following lines:
```Nginx
# Redirect HTTP -> HTTPS

server {
    listen 80;
    server_name www.example.com example.com;

    return 301 https://example.com$request_uri;
}

# Redirect WWW -> NON WWW
server {
    listen 443 ssl http2;
    server_name www.example.com;

    charset utf-8;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    charset utf-8;

    root /var/www/example.com;
    index index.php;

    # SSL parameters
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    # log files
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

}
```

Then:
```sh
chown -R nginx:nginx /var/www/example.com
```

Before restarting the Nginx service test the configuration to be sure that there are no syntax errors:
```sh
nginx -t
```

and we can restart Nginx by typing:
```sh
systemctl restart nginx
```

## Error 413 wordpress
Edit `/etc/nginx/nginx.conf` and add the following line to http context:
```Nginx
client_max_body_size 32m;
```
