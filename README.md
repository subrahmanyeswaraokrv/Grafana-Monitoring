# Grafana-Monitoring
# MongoDB Monitoring with Grafana, Prometheus, and MongoDB Exporter

This guide will help you set up MongoDB monitoring on an Ubuntu server using Grafana for visualization, Prometheus for metric collection, and MongoDB Exporter for exporting MongoDB metrics to Prometheus.

## Author
**Subrahmanyeswarao Karri**

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [1. Install MongoDB Exporter](#1-install-mongodb-exporter)
  - [2. Install Prometheus](#2-install-prometheus)
  - [3. Install Grafana](#3-install-grafana)
- [Configuration](#configuration)
  - [1. Configure MongoDB Exporter](#1-configure-mongodb-exporter)
  - [2. Configure Prometheus](#2-configure-prometheus)
  - [3. Configure Grafana](#3-configure-grafana)
- [Verifying the Setup](#verifying-the-setup)
- [Accessing Grafana Dashboard](#accessing-grafana-dashboard)
- [Troubleshooting](#troubleshooting)

## Prerequisites
Before starting, ensure that you have the following:
- Ubuntu 20.04 or later
- A user with `sudo` privileges
- MongoDB running on the server

## Installation

### 1. Install MongoDB Exporter
MongoDB Exporter is used to collect MongoDB metrics and expose them to Prometheus.

```bash
# Update packages
sudo apt update

# Install required packages
sudo apt install -y golang-go

# Download MongoDB Exporter
cd /opt
sudo wget https://github.com/percona/mongodb_exporter/releases/download/v0.22.0/mongodb_exporter-0.22.0.linux-amd64.tar.gz

# Extract the tar file
sudo tar -xvzf mongodb_exporter-0.22.0.linux-amd64.tar.gz
cd mongodb_exporter-0.22.0.linux-amd64

# Make the exporter executable
sudo chmod +x mongodb_exporter

# Move it to a directory in the $PATH
sudo mv mongodb_exporter /usr/local/bin/

# Verify installation
mongodb_exporter --version
2. Install Prometheus
Prometheus will collect and store the metrics from MongoDB Exporter.

bash
Copy
# Download Prometheus
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v2.39.0/prometheus-2.39.0.linux-amd64.tar.gz

# Extract the tar file
sudo tar -xvzf prometheus-2.39.0.linux-amd64.tar.gz
cd prometheus-2.39.0.linux-amd64

# Move Prometheus binaries to the $PATH
sudo mv prometheus promtool /usr/local/bin/

# Verify Prometheus installation
prometheus --version
3. Install Grafana
Grafana is used for visualizing the metrics collected by Prometheus.

bash
Copy
# Install Grafana
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt update
sudo apt install grafana

# Enable and start Grafana service
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# Verify Grafana is running
sudo systemctl status grafana-server
Configuration
1. Configure MongoDB Exporter
MongoDB Exporter should be configured to point to your MongoDB instance.

bash
Copy
# Start MongoDB Exporter
mongodb_exporter --mongodb.uri="mongodb://<MONGO_USER>:<MONGO_PASSWORD>@localhost:27017" &

# Replace <MONGO_USER> and <MONGO_PASSWORD> with actual MongoDB user credentials
# MongoDB Exporter will start scraping metrics from MongoDB and expose them on port 9216
2. Configure Prometheus
To configure Prometheus to scrape metrics from MongoDB Exporter, you need to modify the Prometheus configuration file.

bash
Copy
# Open Prometheus configuration file
sudo nano /opt/prometheus-2.39.0.linux-amd64/prometheus.yml

# Add the MongoDB Exporter scrape configuration under 'scrape_configs'
scrape_configs:
  - job_name: 'mongodb'
    static_configs:
      - targets: ['localhost:9216']

# Save and exit the editor (Ctrl + X, Y, Enter)
Now, start Prometheus with the following command:

bash
Copy
# Start Prometheus
prometheus --config.file=/opt/prometheus-2.39.0.linux-amd64/prometheus.yml

# Access Prometheus UI at http://localhost:9090
3. Configure Grafana
Access Grafana UI by navigating to http://<YOUR_SERVER_IP>:3000 (default credentials are admin / admin).

Add Prometheus as a data source:

Go to Configuration > Data Sources > Add Data Source.
Select Prometheus.
In the HTTP section, set the URL to http://localhost:9090.
Click Save & Test.
Import a pre-built MongoDB dashboard:

Navigate to Create > Import.
Enter the dashboard ID: 3662 (MongoDB Monitoring).
Click Load and then Import.
Verifying the Setup
Open Prometheus UI at http://<YOUR_SERVER_IP>:9090/targets.
You should see your MongoDB Exporter listed under "Targets" with the status "up."
Access Grafana at http://<YOUR_SERVER_IP>:3000 and navigate to the MongoDB monitoring dashboard to view the metrics.
Accessing Grafana Dashboard
Once Grafana is set up, you can access the dashboard through the web browser by visiting:

cpp
Copy
http://<YOUR_SERVER_IP>:3000
Use the default credentials (admin/admin) to log in.

Troubleshooting
MongoDB Exporter not running: Check the logs with journalctl -u mongodb_exporter and verify that the MongoDB URI is correctly configured.

Prometheus not scraping metrics: Ensure that Prometheus is scraping the MongoDB Exporter endpoint (http://localhost:9216) by verifying the prometheus.yml configuration.

Grafana not displaying data: Ensure that the Prometheus data source is correctly configured and connected.

For more advanced usage or configuration options, refer to the official documentation:

MongoDB Exporter
Prometheus
Grafana
Conclusion
You have successfully set up a MongoDB monitoring stack with Prometheus, Grafana, and MongoDB Exporter on Ubuntu. Now you can easily visualize MongoDB metrics and track performance over time.

Happy monitoring!


This README provides a step-by-step guide to install and configure MongoDB monitoring using Grafana, Prometheus, and MongoDB Exporter on an Ubuntu server. Let me know if you need any more details or adjustments!


