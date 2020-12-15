# Alert Manager

**Overview:** The Alertmanager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver integration such as email, PagerDuty, or OpsGenie (we send the alerts to teams). It also takes care of silencing and inhibition of alerts.

--- 

## Required Background Information

Since we have two Kubernetes clusters, we have two different instance of alert manager configs. The dockerfile copies both configs into the docker container. The configuration file for each Kubernetes deployment is set within the kubernetes deployment file (ie deploy/K8-Blue/prometheus-alerts.yml line 38: ["--config.file=/etc/alert-conf-blue.yml"] )

The rules for what Prometheus should consider an item worthy of alerting is setup within the ./docker/prometheus-docker/rules.yml file.

---

## Read more about alert manager 

- [Alert Manager documentation](https://prometheus.io/docs/alerting/alertmanager/)
  