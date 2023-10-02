# Liferay Installation and Setup on Ubuntu Server

## System Updates

Keeping the system updated:

```bash
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt autoclean
```

### Enable Automatic Security Updates

Install the `unattended-upgrades` by using this command:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## Change hostname

Replace any occurrence of the existing computer name with your new one:

```bash
sudo nano /etc/hostname
sudo nano /etc/hosts
sudo reboot
```

## User

### Create new user

```bash
adduser <username>
```

Add user to `sudo` group:

```bash
usermod -aG sudo <username>
```

### Disable root account

```bash
sudo passwd -l root
```

## SSH

Edit `/etc/ssh/sshd_config`:

```
Port <port>
Protocol 2
PermitRootLogin no
MaxAuthTries 5
PermitEmptyPasswords no
ClientAliveInterval 180
AllowTcpForwarding no
X11Forwarding no
UseDNS no
AllowUsers <user>
```

### SSH Key-Based Authentication

1. Creating SSH Keys
    ```ssh
    ssh-keygen -t RSA -b 4096
    ```

2. Copying an SSH Public Key to Your Server

	Copy the content of your `id_rsa.pub` file to `~/.ssh/authorized_keys` on your remote machine.

### Disabling Password Authentication on your Server

Edit `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
```

## Install webmin

```bash
curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh
sudo sh setup-repos.sh
sudo apt install --install-recommends webmin
```

## Install CSF

```bash
sudo apt install libwww-perl
cd /usr/src/
sudo wget https://download.configserver.com/csf.tgz
sudo tar -xzf csf.tgz
cd csf
sudo sh install.sh
sudo perl /usr/local/csf/bin/csftest.pl
```

## Install jdk

```bash
sudo apt install openjdk-11-jdk
```

## Install postgresql

```bash
sudo apt install postgresql postgresql-contrib postgresql-client postgresql-server-dev-14
sudo systemctl enable --now postgresql.service
git clone https://github.com/teamappir/jalali_utils
cd jalali_utils
sudo make
sudo make install
cd ..
rm -Rf jalali_utils
sudo su -l postgres
createuser --interactive
psql
alter user USER with password 'PASSWORD';
CREATE EXTENSION jalali_utils;
\q
```

Edit `/etc/postgresql/14/main/postgresql.conf` and add `listen_addresses = '*'`.

Edit `/etc/postgresql/14/main/pg_hba.conf` and add `host all all 0.0.0.0/0 md5`.

Restart with `sudo systemctl restart postgresql`.

Check the PostgreSQL port usage: `ss -nlt`


## Webserver

### Install apache2

```bash
sudo apt install apache2
source /etc/apache2/envvars
sudo systemctl enable --now apache2.service
sudo mkdir /var/www/DOMAIN
sudo chown -R www-data:www-data /var/www/DOMAIN
sudo chmod -R 755 /var/www/DOMAIN
```

Create `index.html` with `sudo nano /var/www/DOMAIN/index.html`:

```html
<html>
    <head>
        <title>Welcome to DOMAIN!</title>
    </head>
    <body>
        <h1>Welcome to DOMAIN!</h1>
    </body>
</html>
```

Edit `/etc/apache2/sites-available/000-default.conf`:

```txt
DocumentRoot /var/www/DOMAIN/
```

```bash
sudo systemctl restart apache2.service
```

### Install Cert

#### Manual install

```bash
sudo apt install apache2 certbot python3-certbot-apache
sudo certbot certonly --manual --server https://acme-v02.api.letsencrypt.org/directory --preferred-challenges=dns --email mail --agree-tos -d "*.domain.tld" -d "domain.tld"
```

#### Cloudflare DNS plugin

```bash
sudo snap install core; sudo snap refresh core
sudo apt remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
sudo mkdir /root/.secrets
```

Edit `/root/.secrets/cloudflare.cfg`:

```
dns_cloudflare_email = "email@domain.com"
dns_cloudflare_api_key = "API_KEY" # Global API Key
```

```bash
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.cfg --dns-cloudflare-propagation-seconds 60 --agree-tos -d "*.domain.tld" -d "domain.tld"
```

### Add VHost

Add new conf file: `/etc/apache2/sites-available/DOMAIN.conf`:

```apache
<VirtualHost *:80>
    ServerAdmin admin@DOMAIN
    ServerName DOMAIN
    ServerAlias www.DOMAIN
    DocumentRoot /var/www/DOMAIN
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
	RewriteEngine on
	RewriteCond %{SERVER_NAME} =DOMAIN [OR]
	RewriteCond %{SERVER_NAME} =DOMAIN
	RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

Add new conf file: `/etc/apache2/sites-available/DOMAIN-le-ssl.conf`:

```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin admin@DOMAIN
    ServerName DOMAIN
    ServerAlias www.DOMAIN
    DocumentRoot /var/www/DOMAIN
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
	SSLCertificateFile /etc/letsencrypt/live/DOMAIN/fullchain.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/DOMAIN/privkey.pem
</VirtualHost>
</IfModule>
```

Enable sites config:

```sh
sudo a2enmod rewrite
sudo a2ensite DOMAIN.conf
sudo a2ensite DOMAIN-le-ssl.conf
sudo a2dissite 000-default.conf
sudo systemctl restart apache2
```

### Hardening

Remove Server Version Banner `/etc/apache2/conf-available/security.conf`:

```apache
ServerTokens Prod
ServerSignature Off
```

Enable HTTP/2 module in Apache

```bash
sudo a2enmod http2
```

Install Mod-Security:

```bash
sudo apt install libapache2-mod-security2
sudo a2enmod security2
```

Edit `/etc/apache2/apache2.conf` and add this lines:

```apache
<IfModule security2_module>
    SecRuleEngine on
    ServerTokens Min
    SecServerSignature "Portal"
</IfModule> 
```

Edit `/etc/apache2/mods-available/security2.conf` and comment this line:

```apache
#IncludeOptional /usr/share/modsecurity-crs/*.load
```

## Install liferay

```bash
wget https://github.com/liferay/liferay-portal/releases/download/7.3.7-ga8/liferay-ce-portal-tomcat-7.3.7-ga8-20210610183559721.tar.gz
tar -xvf liferay-ce-portal-tomcat-7.3.7-ga8-20210610183559721.tar.gz
sudo mv liferay-ce-portal-7.3.7-ga8 /opt/
cd /opt
sudo mv liferay-ce-portal-7.3.7-ga8 liferay
cd liferay
sudo mv tomcat* tomcat
sudo useradd liferay -d /opt/liferay/ -b /opt/liferay/ -s /sbin/nologin
sudo chown liferay:liferay liferay -R
```

Create new service with `sudo nano /etc/systemd/system/liferay.service`:

```bash
[Unit]
Description=Liferay Portal
After=network.target

[Service]
Type=forking

ExecStart=/opt/liferay/tomcat/bin/startup.sh
ExecStop=/opt/liferay/tomcat/bin/shutdown.sh

User=liferay
Group=liferay

[Install]
WantedBy=multi-user.target
```

Start service:

```bash
sudo systemctl start liferay.service
```

Run setup wizard `http://DOMAIN:8080`.

After setup finished, stop service and move properties:

```bash
sudo mv /opt/liferay/portal-setup-wizard.properties /opt/liferay/tomcat/webapps/ROOT/WEB-INF/classes/portal-ext.properties
```

Add following lines to `portal-ext.properties`:

```txt
redirect.url.security.mode=domain
redirect.url.ips.allowed=127.0.0.1,SERVER_IP
redirect.url.domains.allowed=DOMAIN

http.header.version.verbosity=off

layout.user.public.layouts.enabled=false
layout.user.private.layouts.enabled=false
```

Install mod_jk:

```bash
sudo apt install libapache2-mod-jk
sudo mv /etc/libapache2-mod-jk/workers.properties /etc/libapache2-mod-jk/workers.properties.bk
```

Create workers.properties with `sudo nano /etc/libapache2-mod-jk/workers.properties`:

```txt
# Define one worker using AJP13
worker.list=worker1

# Set the properties
worker.worker1.type=ajp13
worker.worker1.host=localhost
worker.worker1.port=8009
worker.worker1.secret=SECRT
```

Edit `/opt/liferay/tomcat/conf/server.xml`:

```txt
<Connector protocol="AJP/1.3"
    address="localhost"
    port="8009"
    secretRequired="true"
    secret="SECRET"
    redirectPort="8443" URIEncoding="UTF-8" />
```

Edit `/etc/apache2/sites-available/DOMAIN-le-ssl.conf` file like this:

```apache
JkLogFile		${APACHE_LOG_DIR}/mod_jk.log
JkShmFile		${APACHE_LOG_DIR}/jk-runtime-status
JkLogLevel		error

<IfModule mod_ssl.c>

<VirtualHost *:443>
    ServerAdmin admin@DOMAIN
    ServerName DOMAIN
    ServerAlias www.DOMAIN
    #DocumentRoot /var/www/DOMAIN
    
	JkUnMount /.well-known/* worker1
    JkMount /* worker1
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
	SSLCertificateFile /etc/letsencrypt/live/DOMAIN/fullchain.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/DOMAIN/privkey.pem
	
	SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
	Protocols h2 http/1.1
</VirtualHost>
</IfModule>
```

Restart services:

```bash
sudo systemctl restart apache2
sudo systemctl start liferay
```
