# k3s-Logging with Elasticsearch-Fluentd-Kibana-Logtrail
K3S logs have to be stored in centralized location

ElasticSearch was selected as such centralized location, Kibana is used for data visualisation.

LogTrail is a plugin for Kibana to view, analyze, search and tail log events from multiple hosts in realtime with devops friendly interface inspired by Papertrail.

Fluentd is used as a DaemonSet to ensure we get a running fluentd daemon on each node of the cluster.

# Assumptions

You have a K3S environment

You have namespace created with name infra or you can make the changes in yaml files as per your namespace requirement

# Environment Setup

#### 1. Setup Elasticsearch on your k3s setup

The first node of the cluster we’re going to set up is the master which is responsible for controlling the cluster.

###### Command

$ cd k3s-logtrail-monitoring-EFK-XPACK/elasticsearch/

$ kubectl apply -f es-master-cm.yaml -f es-master-svc.yaml -f es-master-dep.yaml

The second node of the cluster we’re going to set up is the data node that is responsible for hosting the data and executing the queries (CRUD, search, aggregation).

###### Command

$ kubectl apply -f es-data-cm.yaml -f es-data-svc.yaml -f es-data-dep.yaml

The third node client is responsible for exposing an HTTP interface and pass queries to the data node.

###### Command

$ kubectl apply -f es-client-cm.yaml -f es-client-svc.yaml -f es-client-dep.yaml

After a couple of minutes, each node of the cluster should reconcile and the master node should log the following sentence:

###### "Cluster health status changed from [YELLOW] to [GREEN]"

###### Command

$ kubectl logs -f -n infra $(kubectl get pods -n infra | grep elasticsearch-master | sed -n 1p | awk '{print $1}') \
| grep "Cluster health status changed from \[YELLOW\] to \[GREEN\]"

Now,Generate a password and store in a k3s secret.

###### Command

kubectl exec -it $(kubectl get pods -n infra | grep elasticsearch | sed -n 1p | awk '{print $1}') -n infra -- bin/elasticsearch-setup-passwords auto -b

$ kubectl create secret generic elasticsearch-pw-elastic -n infra --from-literal password={elasticsearch-password}

#### 2. Create Docker image of Kibana with logtrail plugin

Create a docker image of kibana with logtrail plugin installed.

###### Command

$ cd k3s-logtrail-monitoring-EFK-XPACK/Docker

$ docker build -t my-logtrail:7.6.2 .

Make sure same image is mentioned in your kibana-dep.yaml

#### 3. Setup Kibana on your k3s setup

Deploy Kibana using below command.

###### Command

$ cd k3s-logtrail-monitoring-EFK-XPACK/kibana

$ kubectl apply -f kibana-cm.yaml -f kibana-svc.yaml -f kibana-dep.yaml -f kibana-ingress.yaml

Login with username elastic and the password (previously generated and stored in a secret).

#### 4. Setup Fluentd on your k3s setup

Create fluentd-config from kubernetes.conf and fluent.conf and then deploy fluentd.

###### Command

$ cd k3s-logtrail-monitoring-EFK-XPACK/fluentd/

$ kubectl -n infra create configmap fluentd-config --from-file kubernetes.conf --from-file fluent.conf

$ kubectl create -f fluentd.yaml

Once fluentd is deployed, check elasticsearch index details using below command. Fluentd should create index with a name "logstash-{date}".

$ curl http://{service-ip}:9200/_cat/indices -u elastic:{password}

#### 5. Create index pattern in kibana for logging

Now our setup is up and running, create an index pattern in kibana. 

###### Go to Management > Kibana > Index Patterns and click on Create index pattern for your index of fluentd. 

You can see an aggregated view of all the logs printed from every pod in logstash-* index. You can filter the logs by any 

attributes attached to the log (for example a Kubernetes label) and navigate over the time.
