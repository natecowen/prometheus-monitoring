# Monitoring Tool 

**Overview:** Prometheus and Grafana are often used to get insight into Kubernetes clusters and containerized applications. Prometheus provides the metrics and Grafana provides a way to visualize these metrics. Common examples include health status of applications, http traffic analysis, insight into memory/cpu utilization, etc. 

This tool is a multipart Kubernetes deployment. It uses Docker images to build Prometheus, Prometheus Extensions, and Grafana. The Docker images are built and deployed to Kubernetes via Jenkins. 

There are several parts to this monitoring system: 
- Prometheus Server: Used to setup rules of what to track, IE Kubernetes, Docker, & other prometheus exporters like blackbox. 
- Prometheus Blackbox Exporter (used to get HTTP Status Codes for our URL endpoints, similar to [Pingdom.com](https://www.pingdom.com/))
- Prometheus Alert Manager (used to process downtime alerts)
- Prometheus Teams Notification (currently using bzon/prometheus-msteams:v1.0.1 to alert to specific Microsoft Teams Channels)
- Grafana (used for dashboards of metrics)
  
> All parts of the system are built with Docker, pushed to the private Docker Registry, and deployed to Kubernetes via Jenkins. **There are two different setups, each setup is specific to a K8 cluster. The green cluster has one more deployment item, blackbox-exporter. It's only part of one cluster to prevent duplicate monitoring of the same resources**

---

## Setup Kubernetes Green

**Create the dev-things Namespace:** Connect to the cluster to verify that the `dev-things` namespace exists on our cluster with this command: ` kubectl get namespaces`. If it does not exists add it via `kubectl create namespace dev-things`

Give the `dev-things` namespace read permissions so that Prometheus can fetch the metrics from Kubernetes API's. Using the `./deploy/setup/clusterRole.yml` file in this repo, create the following role: `kubectl create -f clusterRole.yml`. Verifiy the prometheus role was created with the following command `kubectl get ClusterRoles`.



### Kubernetes Deployments - Green

- **Create Prometheus in K8:** Using the `/deploy/green/prometheus.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prometheus.yml`

- **Create Prometheus BlackBox Exporter in K8:** Using the `/deploy/green/blackbox-exporter.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f blackbox-exporter.yml`

- **Create Prometheus Alerts in K8:** Using the `/deploy/green/prometheus-alerts.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prometheus-alerts.yml`

- **Create Teams Notifications in K8:** Using the `/deploy/green/prom2teams.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prom2teams.yml`

- **Create MongoExporters in K8:** Using the `/deploy/green/mongoexporter.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f mongoexporter.yml`. Repeat for the dev & staging mongo exporters

- **Create Grafana in K8:** Using the `/deploy/green/grafana.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f grafana.yml`

---

## Setup Kubernetes Blue 

Setting up the K8-Blue cluster is very similar to the setup for the K8-Green cluster. The K8-Blue cluster uses the same Docker image for prometheus, but we need to overwrite the prometheus.yaml config file. K8-Blue does not need all of the same settings as the K8-Green. In order to do this we will use a configmap in Kubernetes.

- **Create the dev-things Namespace:** Connect to the cluster to verify that the `dev-things` namespace exists on our cluster with this command: ` kubectl get namespaces`. If it does not exists add it via `kuebectl create namespace dev-things`

- **Create the prometheus configmap:** Ensure you are in the dev-things namespace. Run `kubectl create -f config-map.yaml -n dev-things` to create a Kubernetes config-map & copy the configs for Blue into the container

Give the `dev-things` namespace read permissions so that Prometheus can fetch the metrics from Kubernetes API's. Using the `clusterRole.yml` file in this repo, create the following role: `kubectl create -f clusterRole.yml`. Verifiy the prometheus role was created with the following command `kubectl get ClusterRoles`.

### Kubernetes Deployments - Blue

**Create Prometheus in K8:** Using the `/deploy/blue/prometheus.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prometheus.yml`

**Create Prometheus Alerts in K8:** Using the `/deploy/blue/prometheus-alerts.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prometheus-alerts.yml`

**Create Teams Notifications in K8:** Using the `/deploy/blue/prom2teams.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f prom2teams.yml`

**Create MongoExporters in K8:** Using the `/deploy/green/mongoexporter.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f mongoexporter.yml`. Repeat for the dev & staging mongo exporters

**Create Grafana in K8:** Using the `/deploy/blue/grafana.yml` file in this repo, create the the Kubernetes Service & Deployment: `kubectl apply -f grafana.yml`

---

## Setup The Jenkins Job

After the deployments have been setup in Kubernetes and the Docker metrics have been setup, Jenkins can be setup. All Prometheus items are built from the Jenkinsfile in this repo. Follow the Jenkins setups steps below: 

### JENKINS SETUP

1. Add project type "Multibranch Pipeline"
2. Add source "Git":

   - Project Repository "https://github.com/natecowen/prometheus-monitoring.git"
   - Credentials: JenkinsBot/*****'
   - Behaviors > Filter by name (with regular expression): "^(?!archive|feature|egacy).*$" (don't include quotes)

>**Run the Jenkins Job to confirm everything builds and that the initial versions of the images are deployed.**

---

## Deeper Dive - How Individual Parts Work

1. [Prometheus Server](./docs/01-PrometheusServer.md)
2. [Prometheus Blackbox Exporter](./docs/02-PrometheusBlackBox.md)
3. [Prometheus Alert Manger](./docs/03-AlertManager.md)
4. [Microsoft Teams Notifications](./docs/04-TeamsNotifications.md)
5. [Grafana Dashboards](./docs/05-Grafana.md)

---

### Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/prometheus/latest/getting_started/)
- [Somewhat followed this article](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/). Instead of Config-maps, we map in prometheus.yml and alerts via rule.yml into a custom container that we deploy to our registry. It's less messy than having to update a K8 config map for every change.
- [Alert Manger to Slack/Teams via webhooks](https://www.robustperception.io/using-slack-with-the-alertmanager). Note we also have to use the prom2teams-docker build in this repo as teams expects a very specific format for notifications. [See this repo](https://github.com/bzon/prometheus-msteams).
  