# Webmin

## Install Webmin
Create `/etc/yum.repos.d/webmin.repo` file containing:
```
[Webmin]
name=Webmin Distribution Neutral
#baseurl=https://download.webmin.com/download/yum
mirrorlist=https://download.webmin.com/download/yum/mirrorlist
enabled=1
```

run:
```sh
wget http://www.webmin.com/jcameron-key.asc
rpm --import jcameron-key.asc
yum install webmin
```

## Using letsencrypt certificate for webmin
Add generated cert to webmin cert:
```sh
cat cert.pem > /etc/webmin/miniserv.cert
cat privkey.pem cert.pem > /etc/webmin/miniserv.pem
systemctl restart webmin
```
