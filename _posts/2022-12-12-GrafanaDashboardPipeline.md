---
layout: article
title: Grafana Dashboard Pipeline
author: Jinxiao Yu
tags: Grafana Dashboard Pipeline
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/diving-start.jpg

---

This article will go through the pipeline between NexRes, Prometheus and Grafana.

## NexRes and Prometheus
Prometheus is an open-source monitoring and alerting system that is widely used to monitor the performance of cloud-native applications. The Prometheus server collects metrics from the monitored applications and stores them in a time-series database, allowing users to query the data and create alerts based on specified conditions.

One way to integrate Prometheus with your application is to use a third-party library called prometheus-cpp. This library provides a C++ client for Prometheus that can be used to instrument your application and expose metrics in the Prometheus format. As prometheus-cpp support bazel build tool, this made the integration even easilier since NexRes is also using bazel. 

To allow Prometheus periodically collect the metrics from NexRes and store them in its time-series database, we developed "prometheus_handler" under statitics class to expose metrics in a format that Prometheus can scrape. Each replica is using different port for exposing their metrics, this allow Prometheus to collect all replicas data simultaneously.

<p>
    <img src="{{ site.baseurl }}/assets/images/dashboardPipeline/prometheus_handler.png" alt="Cover photo" style="width: 40%"/>
    <br>
</p>

Once the setup is finished, we need to connect Grafana to Prometheus for display NexRes metrics.

## Grafana and Prometheus
Prometheus and Grafana are two popular tools used for monitoring and observability. Grafana is a visualization tool that can be used to create dashboards and graphs based on the data stored in Prometheus.

To use Prometheus and Grafana together, a common setup is to have Prometheus scraping metrics from various systems and then feeding that data into Grafana for visualization. This setup is often referred to as a "Prometheus-Grafana pipeline". Here is the structure of this pipeline.

<p>
    <img src="{{ site.baseurl }}/assets/images/dashboardPipeline/dashboard_structure.png" alt="Cover photo" style="width: 60%"/>
    <br>
</p>

To create a Prometheus-Grafana pipeline, the first step is to install and configure Prometheus. This typically involves specifying the systems that Prometheus should scrape metrics from and setting up a storage backend for the collected metrics.

Once Prometheus is up and running, the next step is to install and configure Grafana. This involves setting up a data source that points to the Prometheus instance, as well as creating dashboards and graphs to visualize the data.

Once both Prometheus and Grafana are properly configured, they can be used together to monitor and visualize metrics from NexRes. Since all metrics should be properly collected if the NexRes and Prometheus has been connected. Developer can simply using PromQL in Grafana to query NexRes data and build the dashboard.

Please follow this [blog](https://resilientdb.com/blog/2022/12/06/DeployGrafanaDashboardOnOracleCloud.html) to setup Grafana and Prometheus on your system.

## Future Improvements
### Support more time-series databases in NexRes
As the current phase of NexRes, the Prometheus time-series database is integrated. Since Grafana support various type of time-series databases, such as InfluxDB, OpenTSDB, Graphite, etc. To prevent unexpected things and be more user-friendly, we can develop an abstract layer in NexRes to support various databases. For example, there is "prometheus_handler" under the statistics class, we can have another class called "data_handler" and make "prometheus_handler" to be the subclass of "data_handler". Following the same pattern, we can build "influx_handler", and "graphite_handler" under this "data_handler" to support many time-series databases.

### Introduce more performance numbers to the dashboard 
Go though this [google sheet](https://docs.google.com/spreadsheets/d/1a-OGYYoTHMGTZ079Z-bjQP5-aCiXUMHPuszrmkjK7So/edit?usp=sharing), find out the metrics in TODO list that could introduce to NexRes.

### Change embedded Grafana to HTTPs connection
Currently, the Grafana dashboard integrated in nesres.github.io is using http connection. This is a temporality solution that put http connection in https page is a not a secure method. We need to solve this problem in future interation.

### Integrate dashboard into Nexres node manager
While Nexres Node Manager is finished, combine the functionality of cloud deployment installer with node manager.

### Deploy Prometheus configuration by endpoint setup
This improvement also related to node manager. The current Prometheus configuration file only support 4 nexres replicas. If this number increase in the future, the configuration file and Grafana need to adject according to replica number.

### Explore Grafana technique for better visualization
Introduce better visualization to Grafana dashboard.