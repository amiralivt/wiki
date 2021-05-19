# Security Hardening Ubuntu 20.04

## System Updates
Keeping the system updated:
```sh
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
sudo apt-get autoclean
```

### Enable Automatic Security Updates
Install the `unattended-upgrades` by using this command:
```sh
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## User

### Create new user
```sh
adduser <username>
```

Add user to `sudo` group:
```sh
usermod -aG sudo <username>
```

### Disable root account
```sh
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
