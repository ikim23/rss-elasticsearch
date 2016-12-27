# RSS feed watcher using Elasticsearch

### Contents:
- [Install Java 8](#install-java8)

### Requirements:
- Ubuntu Server 16.04.1 LTS
- Oracle Java 8
- Elasticsearch 5.1.1
- Logstash 5.1.1
- X-Pack 5.1.1


## Install Java 8
```
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

## Install Elasticsearch
You can also follow [installation steps](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/deb.html#install-deb) in official documentation.
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.1.1.deb
sudo dpkg -i elasticsearch-5.1.1.deb
```
Start Elasticsearch on system boot:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```
Configure Elasticsearch:
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Add lines to elasticsearch.yml:
```
cluster.name: rss-watcher
node.name: node-1
network.host: 0.0.0.0
```
Start Elasticsearch:
```
sudo systemctl start elasticsearch.service
```

## Install Logstash
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.1.1.deb
sudo dpkg -i logstash-5.1.1.deb
sudo /usr/share/logstash/bin/logstash-plugin install logstash-input-rss
```
Start Logstash on system boot:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable logstash.service
```

## Install X-Pack
Install X-Pack plugin to Elasticsearch:
```
/usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack
```
Add lines to elasticsearch.yml:
```
xpack.security.enabled: false
xpack.monitoring.enabled: false
xpack.graph.enabled: false
xpack.notification.email.account:
    gmail_account:
        profile: gmail
        smtp:
            auth: true
            starttls.enable: true
            host: smtp.gmail.com
            port: 587
            user: myusername@gmail.com
            password: mypassword
```
