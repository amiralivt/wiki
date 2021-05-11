# Wiki.js

## Requirements
Requirements to run Wiki.js are the following:
- Node.js 6.11.1 to 10.x is required.
- MongoDB version 3.2 or later.
- Git version 2.7.4 or later.
- Web Server software such as NGINX, Apache, Caddy, H2O...
- An empty Git repository (optional).
- Minimum of 512MB RAM. 1GB of RAM recommended.
- About 300MB of disk space.
- Domain name with A/AAAA DNS records set up.

## Prerequisites
- A CentOS 7 operating system.
- A non-root user with sudo privileges.

## Initial Steps
```
sudo timedatectl set-timezone 'Asia/Tehran'
sudo yum update -y
sudo yum install -y curl wget vim unzip socat epel-release
```

### Install git
Remove existing git package if installed:
```
sudo yum remove -y git
sudo yum groupinstall -y "Development Tools"
sudo yum install -y gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel curl-devel
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.23.0.tar.gz && tar zxvf git-2.23.0.tar.gz
rm git-2.23.0.tar.gz
cd git-2.23.0
make configure
./configure make prefix=/usr/local all
sudo make prefix=/usr/local install
cd ~
which git
git --version
```

### Install Node.js and npm
```
curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -
sudo yum install -y nodejs
node -v && npm -v
sudo npm install -g npm@latest
```

### Install MongoDB database
Create a `/etc/yum.repos.d/mongodb-org-4.2.repo` file:
```
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

run:
```
sudo yum install -y mongodb-org
sudo systemctl start mongod.service
sudo systemctl enable mongod.service
```

### Install and configure NGINX
```
sudo yum install -y nginx
sudo systemctl enable nginx.service
sudo systemctl start nginx.service
```

Create a `/etc/nginx/conf.d/wiki.js.conf` file:
```
server {

    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    listen [::]:80;
    listen 80;

    server_name example.com;

    charset utf-8;
    client_max_body_size 50M;

    ssl_certificate /etc/letsencrypt/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/example.com/private.key;
    ssl_certificate /etc/letsencrypt/example.com_ecc/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/example.com_ecc/private.key;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_next_upstream error timeout http_502 http_503 http_504;
    }

}
```

Check the NGINX configuration and reload NGINX:
```
sudo nginx -t
sudo systemctl reload nginx.service
```

### Install and setup Wiki.js
Create a document root directory: `sudo mkdir -p /var/www/wikijs`
Change ownership of the `/var/www/wikijs directory` to `your_user`:
```
sudo chown -R [your_user]:[your_user] /var/www/wikijs
```

***NOTE***: Replace your_user in the above command with your non-root user that you should have created as a prerequisite for this tutorial.

From `/var/www/wikijs` directory, run the following command to fetch and install the latest Wiki.js application:
```
cd /var/www/wikijs
curl -sSo- https://wiki.js.org/install.sh | bash
node wiki --version
```

Start the configuration wizard by running:

```
node wiki configure
```

### Setup PM2 Process Manager
```
/var/www/wikijs/node_modules/pm2/bin/pm2 startup
/var/www/wikijs/node_modules/pm2/bin/pm2 save
node wiki restart
pm2 list
```
