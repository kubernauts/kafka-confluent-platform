# Apache Kafka® Confluent Platform on Kubernetes and OpenShift

This repo provides docker files, k8s manifests and OpenShift templates for Apache Kafka® Confluent Platform Community Edition. The reason why we're not using helm charts or the kafka operator at this time is because we like plain manifests which are more suitable for the learning phase.

## Quick start ephemeral

To deploy the ephemeral version for development create the kafka-confluent-5 namespace and deploy the kafka-broker, zookeeper and the related services as follow:

```bash
$ kubectl create -f kafka-confluent-5
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-ephemeral.yaml
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

```bash
$ kubectl create -f kafka-confluent-5
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-persistent-aws-ebs.yaml
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-ephemeral.yaml
```

## Qucik start OpenShift

For OpenShift 3.5 create a project "kafka-confluent-5" in OpenShift and upload the template through OpenShift Webcosole:

https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/openshift/streaming-template-confluent-persistenti-gluster.yaml  

## Testing

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
