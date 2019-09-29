# Kafka loader framework
This is a scalable framework of Kafka producers and consumers that deal with randomly generated messages on a specific topic

## Before you begin

## The producer
```
kubectl apply -f producer/kafka-producer.yaml
```

## The consumer
```
kubectl apply -f consumer/kafka-consumer.yaml
```

## Scaling
```
Example below to scale it up to 10 producers and consumers
kubectl scale sts kafka-producer --replicas=10
kubectl scale sts kafka-producer --replicas=10

```