# Apache Kafka® Confluent Platform on Kubernetes and OpenShift

This repo provides docker files, k8s manifests and OpenShift templates for Apache Kafka® Confluent Platform Community Edition. The reason why we're not using helm charts or the kafka operator at this time is because we like plain manifests which are more suitable for the learning phase.

## Quick start

```bash
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
    
