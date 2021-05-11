# Let's Encrypt

## Wildcard Certificate With Certbot
The command is like this:
```sh
certbot certonly --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory --manual-public-ip-logging-ok -d '*.<your.domain>' -d <your.domain>
```

### Renew Wildcard Certificate With Certbot
Just run:
```sh
certbot certonly --manual --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory -d "*.<your-domain>" -d <your-domain>
```
