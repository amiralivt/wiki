# Install and Configure Elasticsearch

## Install Elasticsearch

Add this to `/etc/hosts`:

```
<this server IP> es-node1
<Liferay server IP> DOMAIN
```

Follow this commands to install Elasticsearch:

```bash
sudo sysctl -w vm.max_map_count=262144
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-kuromoji
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-stempel
```

Edit `/etc/elasticsearch/elasticsearch.yml`:

```
cluster.name: LiferayElasticsearchCluster
discovery.type: single-node
discovery.seed_hosts:
  - es-node1:9300
http.port: 9200
network.host: es-node1
node.name: es-node1
transport.port: 9300
```
	
Add this line to `/etc/elasticsearch/jvm.options`:

```
-Des.enforce.bootstrap.checks=true
```

Add keystore to Elasticsearch:

```bash
sudo echo "keystore_password" > /etc/elasticsearch/pwd
sudo chmod 660 /etc/elasticsearch/pwd
sudo systemctl set-environment ES_KEYSTORE_PASSPHRASE_FILE=/etc/elasticsearch/pwd
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

## Connent Liferay to Elasticsearch

Stop liferay and create new file `/opt/liferay/osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`:

```
productionModeEnabled=B"true"
networkHostAddresses=["http://es-node1:9200"]
```

Start liferay, In the Control Panel, navigate to Configuration → Search and verify the Elasticsearch connection is active.

## Securing Elasticsearch

Copy valid certs to elasticsearch directory:

```bash
sudo mkdir /etc/elasticsearch/certs/
sudo cp privkey.pem /etc/elasticsearch/certs/
sudo cp fullchain.pem /etc/elasticsearch/certs/
```

Add this to `crontab` with `sudo crontab -e`:

```
0 1 * * * cp fullchain.pem /etc/elasticsearch/certs && cp privkey.pem /etc/elasticsearch/certs
```

Edit `/etc/elasticsearch/elasticsearch.yml`:

```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: /etc/elasticsearch/certs/privkey.pem
xpack.security.transport.ssl.certificate: /etc/elasticsearch/certs/fullchain.pem
xpack.security.transport.ssl.certificate_authorities: [ "/etc/elasticsearch/certs/fullchain.pem" ]
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: /etc/elasticsearch/certs/privkey.pem
xpack.security.http.ssl.certificate: /etc/elasticsearch/certs/fullchain.pem
xpack.security.http.ssl.certificate_authorities: [ "/etc/elasticsearch/certs/fullchain.pem" ]
```

Restart `elasticsearch` with `sudo systemctl restart elasticsearch.service`, then set password for default users:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

### Configure a Secure Connection to Elasticsearch in Liferay

Edit `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`:

```
productionModeEnabled=B"true"
networkHostAddresses=["https://DOMAIN:9200"]
authenticationEnabled=B"true"
username="elastic"
password="PASSWORD"
```

## Test connect to elasticsearch

```
curl -u elastic:PASSWORD "https://DOMAIN:9200"
```

## Add self-sign cert to liferay trust

Get cert file:

```
openssl s_client -showcerts -connect DOMAIN:9200 </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > DOMAIN.crt
```

Create truststore file:

```
keytool -import -alias DOMAIN -file /usr/local/share/ca-certificates/DOMAIN.crt -keystore truststore.jks
```

Copy the truststore file to the Liferay directory.

```
cp truststore.jks /opt/liferay/tomcat/bin/
```

Add this lines to `LIFERAY_HOME/tomcat/bin/setenv.sh`:

```
CATALINA_OPTS="$CATALINA_OPTS -Djavax.net.ssl.trustStore=/opt/liferay/tomcat/bin/truststore.jks"
CATALINA_OPTS="$CATALINA_OPTS -Djavax.net.ssl.trustStorePassword=PASSWORD"
```
