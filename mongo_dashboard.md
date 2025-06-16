Subrahmanyeswarao Karri | Data Architect | Database 
==================================================
Mongodb_exporter :
==================
  1  wget https://github.com/percona/mongodb_exporter/releases/download/v0.40.0/mongodb_exporter-0.40.0.linux-amd64.tar.gz
  2  tar -xzf mongodb_exporter-0.40.0.linux-amd64.tar.gz
  3  mv mongodb_exporter-0.40.0.linux-amd64/mongodb_exporter /usr/local/bin/
  4  chmod +x /usr/local/bin/mongodb_exporter
  5  mongodb_exporter --help | grep tls
  6  sha256sum /usr/local/bin/mongodb_exporter
  7  sudo vi /etc/systemd/system/mongodb_exporter.service

[Unit]
Description=MongoDB Exporter
After=network.target

[Service]
User=touadmin
ExecStart=/usr/local/bin/mongodb_exporter \
  --mongodb.uri="mongodb://exporter:xp06tEr@localhost:27017/admin" \
  --compatible-mode \
  --collect-all \
  --collector.collstats \
  --collector.dbstats \
  --collector.topmetrics \
  --collector.indexstats \
  --web.listen-address=":9300"
Restart=always
RestartSec=5
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
Service: 
=========
	sudo systemctl daemon-reload
    sudo systemctl daemon-reexec
	sudo systemctl restart mongodb_exporter
    sudo systemctl status mongodb_exporter

root@dev-shared-database:/mongo/backup/grafana_mongo_prometheus# ps -ef | grep mongo
mongodb   117234       1 12 Jun10 ?        18:35:23 /usr/bin/mongod --config /mongo/mongod.conf
touadmin  194100       1  2 Jun12 ?        02:05:18 /usr/local/bin/mongodb_exporter --mongodb.uri=mongodb://exporter:xp06tEr@localhost:27017/admin --compatible-mode --collect-all --collector.collstats --collector.dbstats --collector.topmetrics --collector.indexstats --web.listen-address=:9300
root      299826  299499  0 17:17 pts/4    00:00:00 grep --color=auto mongo
root@dev-shared-database:/mongo/backup/grafana_mongo_prometheus#

troubleshoot: sudo journalctl -u mongodb_exporter -f

node_exporter:
===============
  1  wget https://github.com/prometheus/node_exporter/releases/download/${LATEST}/node_exporter-${LATEST#v}.linux-amd64.tar.gz
  2  tar -xzf node_exporter-${LATEST#v}.linux-amd64.tar.gz
  3  cd node_exporter-${LATEST#v}.linux-amd64
  4  ./node_exporter &
  5  ps -ef | grep node
  6  ps -ef | grep node
  7  sudo vi /etc/systemd/system/node_exporter.service
  8  sudo cp ~/node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/
  9  systemctl status node_exporter
  10 ps -ef | grep node_exporter
  11 sudo cp ~/node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/
  12 sudo chmod +x /usr/local/bin/node_exporter
  13 sudo chown root:root /usr/local/bin/node_exporter
  14 sudo vi /lib/systemd/system/node_exporter.service"

[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/local/bin/node_exporter --web.listen-address=:9200

[Install]
WantedBy=multi-user.target
  15 sudo systemctl enable node_exporter
  16 sudo systemctl start node_exporter
  17 sudo systemctl status node_exporter
	 root@dev-shared-database:/mongo/backup/grafana_mongo_prometheus# ps -ef | grep node
	 root      299828  299499  0 17:17 pts/4    00:00:00 grep --color=auto node
	 node_ex+ 3723359       1  0 May28 ?        01:08:20 /usr/local/bin/node_exporter --web.listen-address=:9200
  18  vi /etc/systemd/system/node_exporter.service

Prometheus:
==============
wget https://github.com/prometheus/node_exporter/releases/download/${LATEST}/node_exporter-${LATEST#v}.linux-amd64.tar.gz
sudo vi /etc/prometheus/prometheus.yml
global:
  scrape_interval: 30s
  evaluation_interval: 30s

scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['192.XX8.X1.138:9091']

  # MongoDB exporters on various environments
# MongoDB - Performance
  - job_name: 'mongodb_performance'
    static_configs:
      - targets: ['192.XX8.X1.38:9216', '192.XX8.X1.39:9216', '192.XX8.X1.68:9216']
        labels:
          environment: 'performance'
          service: 'mongodb_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.38:9216
        target_label: instance
        replacement: performance-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.39:9216
        target_label: instance
        replacement: performance-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.68:9216
        target_label: instance
        replacement: performance-mongo-n3

# MongoDB - PSP-Base
  - job_name: 'mongodb_psp_base'
    static_configs:
      - targets: ['192.XX8.X1.72:9216', '192.XX8.X1.73:9216', '192.XX8.X1.74:9216']
        labels:
          environment: 'psp-base'
          service: 'mongodb_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.72:9216
        target_label: instance
        replacement: psp-base-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.73:9216
        target_label: instance
        replacement: psp-base-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.74:9216
        target_label: instance
        replacement: psp-base-mongo-n3

# MongoDB - PSP-QA
  - job_name: 'mongodb_psp_qa'
    scrape_interval: 30s
    scrape_timeout: 30s
    static_configs:
      - targets: ['192.XX8.63.110:9216', '192.XX8.63.109:9216', '192.XX8.63.111:9216']
        labels:
          environment: 'psp-qa'
          service: 'mongodb_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.110:9216
        target_label: instance
        replacement: psp-qa-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.109:9216
        target_label: instance
        replacement: psp-qa-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.111:9216
        target_label: instance
        replacement: psp-qa-mongo-n3

# MongoDB - Sales Demo
  - job_name: 'mongodb_sales_demo'
    static_configs:
      - targets: ['192.XX8.X1.94:9216']
        labels:
          environment: 'sales-demo'
          service: 'mongodb_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.94:9216
        target_label: instance
        replacement: psp-sales-demo-mongon1

# MongoDB - Dev
  - job_name: 'mongodb_dev'
    scrape_interval: 40s
    scrape_timeout: 40s
    static_configs:
      - targets: ['192.XX8.X1.165:9300', '192.XX8.X1.185:9300', '192.XX8.X1.188:9300']
        labels:
          environment: 'dev'
          service: 'mongodb_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.165:9300
        target_label: instance
        replacement: dev-mongodb-shared-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.185:9300
        target_label: instance
        replacement: dev-mongodb-shared-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.188:9300
        target_label: instance
        replacement: dev-mongodb-shared-n3

# Node exporters on all the machines
# Node Exporter - Sales Demo
  - job_name: 'node_exporter_sales_demo'
    static_configs:
      - targets: ['192.XX8.X1.94:9200']
        labels:
          environment: 'sales-demo'
          service: 'node_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.94:9200
        target_label: instance
        replacement: psp-sales-demo-mongon1

# Node Exporter - PSP-QA
  - job_name: 'node_exporter_psp_qa'
    static_configs:
      - targets: ['192.XX8.63.109:9200', '192.XX8.63.110:9200', '192.XX8.63.111:9200']
        labels:
          environment: 'psp-qa'
          service: 'node_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.109:9200
        target_label: instance
        replacement: psp-qa-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.110:9200
        target_label: instance
        replacement: psp-qa-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.63\.111:9200
        target_label: instance
        replacement: psp-qa-mongo-n3

# Node Exporter - Dev
  - job_name: 'node_exporter_dev'
    static_configs:
      - targets: ['192.XX8.X1.165:9200', '192.XX8.X1.185:9200', '192.XX8.X1.188:9200']
        labels:
          environment: 'dev'
          service: 'node_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.165:9200
        target_label: instance
        replacement: dev-mongodb-shared-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.185:9200
        target_label: instance
        replacement: dev-mongodb-shared-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.188:9200
        target_label: instance
        replacement: dev-mongodb-shared-n3

# Node Exporter - PSP-Base
  - job_name: 'node_exporter_psp_base'
    static_configs:
      - targets: ['192.XX8.X1.72:9200', '192.XX8.X1.73:9200', '192.XX8.X1.74:9200']
        labels:
          environment: 'psp-base'
          service: 'node_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.72:9200
        target_label: instance
        replacement: psp-base-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.73:9200
        target_label: instance
        replacement: psp-base-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.74:9200
        target_label: instance
        replacement: psp-base-mongo-n3

# Node Exporter - Performance
  - job_name: 'node_exporter_performance'
    static_configs:
      - targets: ['192.XX8.X1.38:9200', '192.XX8.X1.39:9200', '192.XX8.X1.68:9200']
        labels:
          environment: 'performance'
          service: 'node_exporter'
    relabel_configs:
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.38:9200
        target_label: instance
        replacement: performance-mongo-n1
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.39:9200
        target_label: instance
        replacement: performance-mongo-n2
      - source_labels: [__address__]
        regex: 192\.XX8.\.61\.68:9200
        target_label: instance
        replacement: performance-mongo-n3
sudo vi /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9091 \
    --web.enable-admin-api=true
#    --storage.tsdb.retention.time=1d
[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl daemon-reexec
sudo systemctl restart prometheus
sudo systemctl status  prometheus

root@dev-mongodb-poc3:~# ps -ef | grep prometheus
prometh+  917295       1 15 Jun12 ?        14:32:55 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries --web.listen-address=0.0.0.0:9091 --web.enable-lifecycle
root     1030831 1030163  0 17:36 pts/1    00:00:00 grep --color=auto promet

URL:
curl http://192.XX8.X1.165:9300/metrics
curl -X POST http://localhost:9091/api/v1/admin/tsdb/delete_series -d 'match[]=mongodb_up{instance="psp-sales-demo-mongodb"}'

Troubleshoot:
=============
 1 nc -vz 192.XX8.X1.185 9300
 2 sudo ufw status
 3 telnet 192.XX8.X1.185 9300
 4 sudo ufw allow 9300/tcp
 5 telnet 192.XX8.X1.38 9216
 6 sudo journalctl -u prometheus -b --no-pager | grep enable-admin-api
 7 promtool check config /etc/prometheus/prometheus.yml
 
Grafana:
============
root@dev-mongodb-poc3:~# ps -ef | grep grafana
grafana     1088       1  0 Apr25 ?        01:58:17 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/log/grafana cfg:default.paths.data=/var/lib/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins cfg:default.paths.provisioning=/etc/grafana/provisioning
grafana     1126    1088  0 Apr25 ?        00:03:24 /var/lib/grafana/plugins/yesoreyeram-infinity-datasource/gpx_infinity_linux_amd64
root     1031389 1030163  0 17:51 pts/1    00:00:00 grep --color=auto grafana

✅ Step 1: Update Your System
bash
Copy
Edit
sudo apt update && sudo apt upgrade -y
✅ Step 2: Install Required Dependencies
bash
Copy
Edit
sudo apt install -y software-properties-common
✅ Step 3: Add Grafana APT Repository
bash
Copy
Edit
sudo apt install -y apt-transport-https
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list > /dev/null
✅ Step 4: Install Grafana
bash
Copy
Edit
sudo apt update
sudo apt install grafana -y
✅ Step 5: Enable and Start Grafana Service
bash
Copy
Edit
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
Check Grafana status:

bash
Copy
Edit
sudo systemctl status grafana-server
✅ Step 6: Access Grafana Web UI
Open your browser and go to:

cpp
Copy
Edit
http://<your_server_ip>:3000
Default Username: admin

Default Password: admin (You’ll be prompted to change it at first login)

✅ (Optional) Open Port 3000 in Firewall
If you use UFW:

bash
Copy
Edit
sudo ufw allow 3000/tcp
