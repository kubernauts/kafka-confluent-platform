# Apache Kafka® Confluent Platform on Kubernetes and OpenShift

This repo provides docker files, k8s manifests and OpenShift templates for Apache Kafka® Confluent Platform Community Edition. The reason why we're not using helm charts or the kafka operator at this time is because we like plain manifests which are more suitable for the learning phase.

## Quick start

```bash
$ kubectl create -f https://raw.githubusercontent.com/kubernauts/kafka-confluent-platform/master/k8s/streaming-ephemeral.yaml
```    
