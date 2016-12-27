# Watch RSS feed using Elasticsearch
This article show you how to setup Elasticsearch to search through RSS feed and send e-mail notification on match. This can be useful for example in second hand forums/sites. (Example uses site: [Bazos.sk](https://www.bazos.sk/))

### Requirements:
- Ubuntu Server 16.04.1 LTS
- Oracle Java 8
- Elasticsearch 5.1.1
- Logstash 5.1.1
- X-Pack 5.1.1

### Contents:
- [Install Java 8](#install-java-8)
- [Install Elasticsearch](#install-elasticsearch)
- [Install Logstash](#install-logstash)
- [Install X-Pack](#install-x-pack)

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
Create mapping for RSS feed:
```
curl -XPUT "http://localhost:9200/bazos" -d'
{
   "settings": {
      "index": {
         "number_of_shards": 1,
         "number_of_replicas": 0
      },
      "analysis": {
         "analyzer": {
            "folding": {
               "type": "custom",
               "tokenizer": "standard",
               "char_filter": ["html_strip"],
               "filter": ["lowercase", "asciifolding"]
            }
         }
      }
   },
   "mappings": {
      "feed": {
         "_all": {
            "enabled": false
         },
         "properties": {
            "feed": {
               "type": "keyword"
            },
            "link": {
               "type": "keyword"
            },
            "published": {
               "type": "date"
            },
            "message": {
               "type": "string",
               "analyzer": "folding"
            },
            "title": {
               "type": "string",
               "analyzer": "folding"
            }
         }
      }
   }
}'
```

## Install Logstash
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.1.1.deb
sudo dpkg -i logstash-5.1.1.deb
sudo /usr/share/logstash/bin/logstash-plugin install logstash-input-rss
```
Create Logstash configuration file:
```
sudo nano /etc/logstash/conf.d/logstash.conf
```
Insert lines to logstash.conf:
```
input {
	rss {
		interval => 300
		url => "https://www.bazos.sk/rss.php?rub=PC"
	}
}

filter {
	mutate {
		remove_field => [
			"@timestamp",
			"@version",
			"author",
			"tags"
		]
		rename => {
			"Feed" => "feed"
		}
	}
}

output {
	elasticsearch {
		hosts => [ "localhost" ]
		index => "bazos"
		document_type => "feed"
		document_id => "%{link}"
	}
}
```
This configuration downloads and index new data to Elasticsearch every 5 minutes.

Start Logstash on system boot:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable logstash.service
```
Start Logstash:
```
sudo systemctl start logstash.service
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
See [Gmail setup](https://www.elastic.co/guide/en/x-pack/5.1/actions-email.html#gmail) in official documentation. Don't forget to allow access to your Gmail account for less secure applications.

Restast Elasticsearch to accept configuration changes :
```
sudo systemctl restart elasticsearch.service
```
Create watcher for RSS feed:
```
curl -XPUT "http://localhost:9200/_xpack/watcher/watch/match_in_bazos" -d'
{
   "trigger": {
      "schedule": {
         "interval": "5m"
      }
   },
   "input": {
      "search": {
         "request": {
            "indices": ["bazos"],
            "body": {
               "_source": ["title"],
               "query": {
                  "bool": {
                     "should": [
                        {"match": { "title": "intel i7" }},
                        {"match": { "message": "intel i7" }}
                     ],
                     "filter": {
                        "range": {
                           "published": {
                              "from": "{{ctx.trigger.scheduled_time}}||-5m",
                              "to": "{{ctx.trigger.triggered_time}}"
                           }
                        }
                     }
                  }
               },
               "sort": [{"published": { "order": "desc" }}]
            }
         }
      }
   },
   "condition": {
      "compare": {
         "ctx.payload.hits.total": {
            "gt": 0
         }
      }
   },
   "actions": {
      "email_admin": {
         "email": {
            "to": "username@gmail.com",
            "subject": "RSS Watcher",
            "body": "{{ctx.payload.hits.total}} matches found.\n\nSee:\n{{ctx.payload.hits.hits}}"
         }
      }
   }
}'
```
This watcher runs query every 5 minutes and sends email on match (searches for "intel i7" in title and message of feed).
