---
layout: article
title: NexRes Grafana Dashboard Installation
author: Jinxiao Yu
tags: NexRes Grafana Dashboard Installation
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/codecover.jpg

---

Nexres Grafana Dashboard is a Grafana base dashboard for Nexres. It aims to provide a simple real-time interface for developers to monitor and diagnose Nexres. The data is stored in the Prometheus time-series database and queried by Grafana using PromeQL. The system usage data is provided by Prometheus third-party exporter Node Exporter.

# Requirements
Using Nexres dynamic dashboard requires the installation of Prometheus, Node Exporter, Prometheus-cpp, and Grafana. It has been successfully tested with Bazel building in C++17 on Ubuntu 20.04.4 LTS (Windows 11 WSL) and Visual Studio Code.

# Installation
## Install Prometheus
go to https://prometheus.io/download/ download prometheus
```
tar xvfz (your-prometheus-tar-file)
```

## Install node_exporter
go to https://github.com/prometheus/node_exporter download the lastest version of node_exporter from release
```
tar xvfz (your-node-exporter-tar-file)
```

## Install grafana
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt update 
sudo apt install grafana
```

## Start grafana
```
sudo service grafana-server start
sudo service grafana-server enable
```

Default grafana port is 3000



# Start the dashboard
## Prometheus
Default prometheus port is 9090
```
./prometheus
```

## Node_exporter
Default node exporter port is 9100
```
./node_exporter
```

# Local Test
1. Change the exporter endpoint from the prometheus config ([example](https://github.com/resilientdb/resilientdb/blob/master/monitoring/prometheus/prometheus.yml)) to the localhost:port
   port number can be found from the [script](https://github.com/resilientdb/resilientdb/blob/master/service/tools/kv/server_tools/start_kv_service_monitoring.sh) (e.g. 8090)

2. Start prometheus, grafana, and node_exporter

3. Start the KV service using the [script](https://github.com/resilientdb/resilientdb/blob/master/service/tools/kv/server_tools/start_kv_service_monitoring.sh):  service/tools/kv/server_tools/start_kv_service_monitoring.sh
   
5. Open the prometheus to check the service status: http://localhost:9090/targets?search=

6. Set up your grafana dashboard from http://locahost:3000/
   A grafana dashboard template can be found [here](https://github.com/resilientdb/resilientdb/blob/master/documents/file/Nexres-1654906717062.json).

# Remote Deploy
1. Set the exporter endpoint from the prometheus config with each deploy node ([example](https://github.com/resilientdb/resilientdb/blob/master/monitoring/prometheus/prometheus.yml))
   
2. Set node_exporter in each of the deploy node

3. Set the grafana export point in the deploy script [scripts/deploy/script/deploy.sh#L21](https://github.com/resilientdb/resilientdb/blob/master/scripts/deploy/script/deploy.sh#L21 )
   
5. Deploy the nodes
   
