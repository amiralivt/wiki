# CSF

## Install
```sh
yum install perl-libwww-perl net-tools perl-LWP-Protocol-https perl-GDGraph
cd /usr/src
rm -fv csf.tgz
wget https://download.configserver.com/csf.tgz
tar -xzf csf.tgz
cd csf
sh install.sh
```

### Test
```sh
perl /usr/local/csf/bin/csftest.pl
```
