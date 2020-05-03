# k3s-Logging with Elasticsearch-Fluentd-Kibana-Logtrail
K3S logs have to be stored in centralized location

ElasticSearch was selected as such centralized location, Kibana is used for data visualisation.

LogTrail is a plugin for Kibana to view, analyze, search and tail log events from multiple hosts in realtime with devops friendly interface inspired by Papertrail.

Fluentd is used as a DaemonSet to ensure we get a running fluentd daemon on each node of the cluster.

# Assumptions

You have a K3S environment

You have namespace created with name infra or you can make the changes in yaml files as per your namespace requirement
