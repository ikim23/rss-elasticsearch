# RSS feed watcher using Elasticsearch

### Contents:
- [Install Java 8](#install-java8)

### Requirements:
- Ubuntu Server 16.04.1 LTS
- Oracle Java 8
- Elasticsearch 5.1.1


## Install Java 8
```
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

## Install Elasticsearch
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
Add lines:
```
cluster.name: rss-watcher
node.name: node-1
network.host: 0.0.0.0
```
```
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```
