# Apache Kafka® Confluent Platform on Kubernetes and OpenShift

This repo provides docker files, k8s manifests and OpenShift templates for Apache Kafka® Confluent Platform Community Edition. The reason why we're not using helm charts or the kafka operator at this time is because we like plain manifests which are more suitable for the learning phase.

## Prerequisites

This implementation was developed and tested on EKS with k8s 1.10.3, with tk8 made k8s 1.11.2 and OpenShift 3.5 and 3.7.
If one would like to get it working on k8s < 1.9.x, the `apiVersion` for StatefulSets should be changed to `apps/v1beta1` or `apps/v1beta2` instead of `apps/v1`.

## Quick start ephemeral

To deploy the ephemeral version for development create the kafka-confluent-5 namespace and deploy the kafka-broker, zookeeper and the related services as follow:

```bash
$ kubectl create ns kafka-confluent-5
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-ephemeral.yaml -n kafka-confluent-5
```

```bash
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
kafka-0   1/1       Running   0          1m
kafka-1   1/1       Running   0          1m
kafka-2   1/1       Running   0          1m
zoo-0     1/1       Running   0          1m
zoo-1     1/1       Running   0          1m
zoo-2     1/1       Running   0          1m
```

## Quick start persistent

To deploy the persistent version with PVCs, create the kafka-confluent-5 namespace, create the storage class gp2 (for aws-ebs here) and deploy the kafka-broker, zookeeper and the related services with persitent support as follow: 

Please adapt the zone in the storage class where you've your k8s cluster:

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  labels:
    kubernetes.io/cluster-service: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: "us-east-1a"
```

And run the follwoing commads:

```bash
$ kubectl create ns kafka-confluent-5
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/storageclass-aws-ebs.yaml
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-persistent-aws-ebs.yaml -n kafka-confluent-5
```

## Qucik start OpenShift

For OpenShift 3.5 create a project "kafka-confluent-5" in OpenShift and upload the template through OpenShift Webcosole:

https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/openshift/streaming-template-confluent-persistenti-gluster.yaml  

## Testing with Confluent Test Client

Deploy confluent test-client

```bash
kubectl create -f confluent-testclient-confluent-5.yaml
```

### Create topics

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-topics --zookeeper zoo --topic topic1 --create --partitions 30 --replication-factor 3
```

### List topics

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-topics --zookeeper zookeeper:2181 --list
```

### Describe topics

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-topics --zookeeper zookeeper:2181 --describe
```

### Produce messages

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-console-producer --broker-list broker:9092 --topic topic1
```

### Consume messages

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-console-consumer --bootstrap-server broker:9092 --topic topic1 --from-beginning
```

### Delete topic

```bash
kubectl exec -it confluent-client -- /usr/bin/kafka-topics --zookeeper zookeeper:2181 --delete --topic topic1
```

### Testing with Kafkacat

Deploy Kafkacat:

```bash
kubectl create -f kafkacat.yml
```

Exec into the producer container and write messages to topic1:

```bash
$ k exec -it kafkacat-12345 -c producer /bin/bash
$ for i in `seq 1 100`; do  echo "hello kafka world"  | kafkacat -b kafka:9092 -t topic1; done
```

In another shell exec into the cosumer container and read messages:

```bash
$ k exec -it kafkacat-12345 -c consumer /bin/bash
# kafkacat -C -b kafka:9092 -t topic1
```

## Deploy Yahoo Kafka Manager with Helm

Make sure your k8s cluster is properly configured for Helm:

```bash
helm init
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

Clone the great implementation of Charles Martinot for the Yahoo Kafka Manager: 

```bash
$ git clone https://github.com/MacTynow/kafka-manager-chart.git
$ cd kafka-manager-chart 
```

Adapt values.yaml

```bash
  useKafkaZookeeper: true
  zkHosts: "zoo"
```

Install Yahoo Kafka Manager with Helm:

```bash
$ helm install .
```

Follow the instructions of the helm output, and access the GUI, provide "zoo" as the Cluster Zookeeper Hosts (the headless service).

## Rancher integration

Deploy Rancher on your k8s cluster with a single command:

```bash
kubectl create -f https://raw.githubusercontent.com/kubernauts/tk8/master/addons/rancher/master.yaml
```

Rancher will be up in 2 minutes, you can get the LB address through:

```bash
$ kubectl get svc -n cattle-system
```

Add a project in Rancher by navigating to "Projects/Namespaces" in Rancher UI, e.g. "confluent-kafka-5-project" and move the namespace "confluent-kafka-5" namespace to the newly created project "confluent-kafka-5-project". With this you can add new users to the project and have control! 

## Monitoring and Alerting with Prometheus and Grafana through Rancher

Select your newly created project "confluent-kafka-5-project", navigate to Catalog Apps in the nice menu bar of Rancher and add Prometheus from Rancher's Library Catalog with the follwoing settings:

### PROMETHEUS SERVER

```bash
Expose Prometheus using Layer 7 Load Balancer: false
Prometheus Service Type: ClusterIP
Prometheus Persistent Volume StorageClass: gp2 (whic was created through the storageclass on EKS)
Create Persistent Volume for Prometheus: true
```

### GRAFANA SETTINGS

```bash
Expose Grafana using Layer 7 Load Balancer: false
Grafana Service Type: NodePort
Grafana Persistent Volume Enabled: true
Storage Class for Grafana: gp2
```
### ALERTMANAGER

```bash
Expose Alertmanager using Layer 7 Load Balancer: false
Alertmanager Service Type: ClusterIP
Create Persistent Volume for Alertmanager: true
Alertmanager Persistent Volume StorageClass: gp2
```

And click on launch, after 2 minutes you shall get:

```bash
$ k get pods -n prometheus
NAME                                             READY     STATUS    RESTARTS   AGE
prometheus-alertmanager-6df98765f4-f79kh         2/2       Running   0          3m
prometheus-grafana-5bf7b6d949-pldn7              2/2       Running   0          3m
prometheus-kube-state-metrics-6584885ccf-l7tkm   1/1       Running   0          3m
prometheus-node-exporter-5qvjc                   1/1       Running   0          3m
prometheus-node-exporter-bsp48                   1/1       Running   0          3m
prometheus-node-exporter-f64cr                   1/1       Running   0          3m
prometheus-node-exporter-r7t7c                   1/1       Running   0          3m
prometheus-node-exporter-stdjg                   1/1       Running   0          3m
prometheus-server-5959898967-s6hjd               2/2       Running   0          3m
```

Run something like this to access the Grafana Dashboard:

```bash
$ kubectlforward prometheus-grafana-5bf7b6d949-pldn7 3000
```

Import the kafka-overview_rev1.json and grafana-kafka-dashboard.json provided in the root of this repo as "Kafka Overview" Dashboard.
You'll get a bunch of other very useful dashboard provided throught the Rancher installation.

### Clean up

To clean up your environment, run:

```bash
$ k delete all -l app=kafka -n kafka-confluent-5
$ k delete pvc --all
$ k delete ns kafka-confluent-5
```

### Screenshots

![Alt text](./rancher.png?raw=true "Rancher")
![Alt text](./grafana-kafka-overview.png?raw=true "Grafana Kafka Overview")
![Alt text](./grafana-dashboards.png?raw=true "Grafana Kafka Dashboards")
![Alt text](./grafana-kafka-dashboard.png?raw=true "Grafana Kafka Dashboard")
![Alt text](./openshift.png?raw=true "OpenShift Dashboard")


### Useful links

https://github.com/edenhill/kafkacat

https://github.com/Yolean/kubernetes-kafka

https://docs.confluent.io/current/app-development/kafkacat-usage.html

https://docs.confluent.io/current/installation/installing_cp/cp-helm-charts/docs/index.html#cp-helm-quickstart

https://hackernoon.com/a-blockchain-experiment-with-apache-kafka-97ee0ab6aefc

https://hackernoon.com/simple-chatops-with-kafka-grafana-prometheus-and-slack-764ece59e707

### Get in touch

Join us on Kubernauts Slack Channel:

https://kubernauts-slack-join.herokuapp.com/


