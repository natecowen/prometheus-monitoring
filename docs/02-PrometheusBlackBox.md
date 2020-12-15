# Blackbox Exporter

**Overview:** The blackbox exporter allows blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP and ICMP. We use this as an internal ping to our https endpoints, similar to how pingdom works.

--- 

## Parts That Make Our Setup Work

Blackbox exporter is setup in two areas. The dockerfile copies over a basic blackbox.yml file that sets up the service. This is where you setup the type of probing you would like, IE http/s, tcp, etc. 

The second part of the configuration is done in the prometheus config file. The prometheus config file tells blackbox what endpoints to check and where our blackbox instance is located. 

## Link to Blackbox Exporter

- [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter)
