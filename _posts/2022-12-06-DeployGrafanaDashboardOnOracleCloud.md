---
layout: article
title: Deploy Grafana Dashboard On Oracle Cloud
author: Jinxiao Yu
tags: Deploy Grafana Dashboard On Oracle Cloud
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

This article will guide you on deploying NexRes Grafana Dashboard on an Oracle Cloud instance. Since Prometheus and node exporter cannot directly install using apt-get, installing these components is complicated and tedious. NexRes_Grafana_Installer provides an effortless experience to automatically deploy all the components for NexRes Grafana Dashboard on Oracle Cloud. Now let us dive in!

## Requirements
This installer only tested on Oracle Cloud instance in ubuntu 20.04. Please adjust the script if you want to run it on AWS or local. Besure nexres is already installed on your instance and fully tested.

## Clone nexres_grafana_installer 
Clone the repositry in your home directry
```
https://github.com/jyu25utk/nexres_grafana_installer
```

## Adjust start_nexres.sh
In default, if you cloned resilientdb repositry (nexres) in your home directry. The full directry address should be "/home/ubuntu/resilientdb". If your nexres working directry is same as I mentioned above, jump to next step.

Otherwise, modify the "start_nexres.sh" file before you process to the next step. Change the following pathes according to your nexres configuration.
```
SERVER_PATH=/home/ubuntu/resilientdb/bazel-bin/kv_server/kv_server
SERVER_CONFIG=/home/ubuntu/resilientdb/example/kv_config.config
WORK_PATH=/home/ubuntu/resilientdb
```

## Install!
Simply run command
```
sudo sh install_dashboard.sh
```
All components should automaticly install to your system!

Make sure to restart your instance using 
```
sudo reboot
```

## Test your dashboard
Find your public ip address, access your grafana interface by using broswer go to

 > (public ip address):3000

Setup your grafana dashboard as same as the local one

Access prometheus management interface by using broswer go to 

> (public ip address):9090