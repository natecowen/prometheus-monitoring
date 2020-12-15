# Prometheus Server

**Overview:** We use the official Docker Hub Image for Prometheus Server. This allows us to easily build our own image with it as a starting point. 

--- 

## Required Background Information

Prometheus Server is a pull based system. It goes to an endpoint and scrapes metrics based on a set time interval. It's ideal for monitoring container based applications and has built in support from both Docker and Kubernetes. 

Prometheus data is not sacred, it is only kept for 7 days in our setup. This is set in the `deploy/promtheus.yml` file with the line `"--storage.tsdb.retention=7d"`. Long term metrics should be captured via an actual logging system (ie Elk stack). Prometheus is only used to view health of apps, current utilization, to verify endpoints are up, or jobs have been ran successfully. 

Prometheus can use self-discovery for Kubernetes metrics. This is due to the fact that it is running as a pod on Kubernetes. If it's ran on a different server than the K8 cluster, then it needs to have the metrics endpoint exposed outside of the cluster.

Out of the box, Prometheus doesn't do a lot. It's meant to scrape endpoints, the type of endpoints it can scrape can be extended with exporters. Exporters typically report metrics via a /metrics endpoint that they expose. Prometheus scrape those endpoints. 

---

## Parts That Make Our Setup Work

The K8 deployment sets options like where the config file is located, path to storing data, and data retention. It also exposes standard K8 items like a Nodeport so the container can be accessed via a URL.

The Dockerfile is used to build our custom image. It just maps in our config file and the alert rules. 

**Prometheus Configuartion:** Configuration is set via the prometheus.yml file. This file sets the targets (ipaddress & port) that prometheus will scrape (ex kubernetes or blackbox exporter). It then self discovers items within those targets (ex kubernetes pods). It uses labels to mark which items are to be kept and scraped. [Get more info on configuration here](https://prometheus.io/docs/prometheus/latest/configuration/configuration/).  

> Just because items are discovered for jobs, it doesn't mean that Prometheus will scrape them. For example, Prometheus drops all labels that start with two underscores. Kubernetes labels all start with two underscores. In order to capture only what you want, the config file uses relabel_configs to capture the targets you want. [Read more about relabeling here](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config). It's hard to understand, but vital to Prometheus and our setup.

**Alert Rules:** Alert rules are additional configs used to send alerts to the alert manager docker container. The alert manager then is setup to push alerts to services such as SMTP or Microsoft Teams. Currently, the only alert setup is for HTTP Statuses other than 200 for API URL's and Production Microsites (HTTPS status is gathered via Blackbox Exporter).