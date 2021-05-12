# Webmin

## Install Webmin
Edit the `/etc/apt/sources.list` file on your system and add the line:
```
deb https://download.webmin.com/download/repository sarge contrib
```

run:
```sh
cd /root
wget https://download.webmin.com/jcameron-key.asc
apt-key add jcameron-key.asc

apt-get install apt-transport-https
apt-get update
apt-get install webmin

```
