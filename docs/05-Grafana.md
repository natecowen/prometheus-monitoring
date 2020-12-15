# Grafana

**Overview:** Grafana provides charts based on the data it reads from Prometheus. Grafana is often paired with items like ELK or Prometheus to display data in a more meaningful manner.  

--- 

## Required Background Information

Most of our dashboards are modified versions of [community designed dashboards](https://grafana.com/dashboards). Our dockerfile copies over our dashboards and configs to the appropriate locations within the grafana container. 

To modify these dashboard you will need to do the following: 
- [Login to grafana via the default account](https://developer.something.com:30004). The login botton is on the lower portion of the side menu. 
- Select the dashboard that you would like to view/edit from the left hand navigation
- Edit the dashboard as you see fit. 
- Click the save button on the top of the dashboard. 
- Copy the json output and update the json-dashboard within '../docker/grafana-docker/json-dashboards/'
- Push your changes to github and kickoff a build within the Jenkins pipeline. This will automatically deploy the new dashboard. 

---

## Read more about Grafana

- [Grafana](https://grafana.com/)
- [Search Pre-made Grafana Dashboards](https://grafana.com/dashboards)
  
  