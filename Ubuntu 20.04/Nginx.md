# Nginx
## Install
```sh
sudo apt install nginx
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

## Setting Up Server Blocks
Create the directory for **your_domain** as follows, using the `-p` flag to create any necessary parent directories:
```sh
sudo mkdir -p /var/www/your_domain/html
```

Next, ownership and permissions :
```sh
sudo chown -R www-data:www-data /var/www/your_domain/html
sudo chmod -R 755 /var/www/your_domain
```

Next, create a sample `index.html` page:
```sh
nano /var/www/your_domain/html/index.html
```

Inside, add the following sample HTML:
```html
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
```

Create new file `/etc/nginx/sites-available/your_domain` and add following lines:
```nginx
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Next, let’s enable the file by creating a link from it to the `sites-enabled` directory, which Nginx reads from during startup:
```sh
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Edit `/etc/nginx/nginx.conf` file:
```nginx
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```

Test syntax errors and restart Nginx:
```sh
sudo nginx -t
sudo systemctl restart nginx
```

## Secure Nginx with Let's Encrypt

### Installing Certbot
```sh
sudo apt install certbot python3-certbot-nginx
```

### Obtaining an SSL Certificate
```sh
sudo certbot --nginx -d example.com -d www.example.com
```
