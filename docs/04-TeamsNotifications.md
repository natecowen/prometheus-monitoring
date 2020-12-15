# Team Notifications

**Overview:** We use a third party docker image called prometheus-mstreams. It's a lightweight go application that pushes to our Microsoft teams via a webhook. 

--- 

## Required Background Information

There is little to no setup for this application to work. All of the setup is completed via the Kubernetes deployment file (ie '../deploy/K8-Blue/prom2teams.yml') and the alert-manager config file (ie '../docker/alert-manager/alert-conf-blue.yml'). It pushes all notification via a webhook that it curls an alert to.

---

## Read more about alert manager 

- [prometheus-mstreams Github](https://github.com/bzon/prometheus-msteams)
- [Setting up a teams webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/connectors/connectors-using#setting-up-a-custom-incoming-webhook)
  