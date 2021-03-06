### NodeJS

```bash
curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### PM2

```bash
sudo npm install pm2@latest -g
sudo pm2 startup
```

### Elasticsearch Installation

```bash
sudo apt update
sudo apt install openjdk-11-jre-headless
mkdir hyperion
cd hyperion
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-beta1-amd64.deb
sudo apt install ./elasticsearch-7.0.0-beta1-amd64.deb
```

Edit `/etc/elasticsearch/elasticsearch.yml`

```
cluster.name: myCluster
node.attr.box_type: hot
bootstrap.memory_lock: true
```

Edit `/etc/elasticsearch/jvm.options`
```
# Set your heap size, avoid allocating more than 31GB, even if you have enought RAM.
-Xms16g
-Xmx16g
```

Disable swap on /etc/fstab

Allow memlock:
run `sudo systemctl edit elasticsearch` and add the following lines


```
[Service]
LimitMEMLOCK=infinity
```

Start elasticsearch and check the logs (verify if the memory lock was successful)

```bash
sudo service elasticsearch start
sudo less /var/log/elasticsearch/myCluster.log
sudo systemctl enable elasticsearch
```

Test the REST API `curl http://localhost:9200`

```
{
  "name" : "igorls-dev-server",
  "cluster_name" : "myCluster",
  "cluster_uuid" : "UgZisGJTTgKq1Q_kug70dg",
  "version" : {
    "number" : "7.0.0-beta1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "15bb494",
    "build_date" : "2019-02-13T12:30:14.432234Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.7.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### Kibana Installation

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.0.0-beta1-amd64.deb
sudo dpkg -i kibana-7.0.0-beta1-amd64.deb
sudo systemctl enable kibana
```

Open Kibana on `http://localhost:5601`

### RabbitMQ Installation

```bash
sudo apt install rabbitmq-server
sudo rabbitmq-plugins enable rabbitmq_management
sudo rabbitmqctl add_vhost /hyperion
sudo rabbitmqctl add_user my_user my_password
sudo rabbitmqctl set_user_tags my_user administrator
```

Check access to the WebUI `http://localhost:15672`

Remember to allow your user RW access to the `/hyperion` vhost

### Redis Installation

```bash
sudo apt install redis-server
```

Edit `/etc/redis/redis.conf` and change supervised to `systemd`

```bash
sudo systemctl restart redis.service
```

### Hyperion Indexer

```bash
git clone https://github.com/eosrio/Hyperion-History-API.git
cd Hyperion-History-API
npm install
cp example-ecosystem.config.js ecosystem.config.js
```

Edit `ecosystem.config.js`


### Setup Indices and Aliases

Load templates first by starting the Hyperion Indexer in preview mode `PREVIEW: 'true'`

Then on kibana dev console create the initial indices and aliases
```
PUT mainnet-action-v1-000001
PUT mainnet-abi-v1-000001
PUT mainnet-block-v1-000001

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "mainnet-abi-v1-000001",
        "alias": "mainnet-abi"
      }
    },
    {
      "add": {
        "index": "mainnet-action-v1-000001",
        "alias": "mainnet-action"
      }
    },
    {
      "add": {
        "index": "mainnet-block-v1-000001",
        "alias": "mainnet-block"
      }
    }
  ]
}
```

Before indexing actions into elasticsearch its required to do a ABI scan pass (its very fast - 45 million blocks in 2 hours)

Start with
```
ABI_CACHE_MODE: true,
FETCH_BLOCK: 'false',
FETCH_TRACES: 'false',
FETCH_DELTAS: 'true'
```

