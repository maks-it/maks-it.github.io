# Kubernetes Setup Redis

## Real World Example: Configuring Redis using a ConfigMap

Follow the steps below to configure a Redis cache using data stored in a ConfigMap.

First create a ConfigMap with an empty configuration block:

```bash
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
data:
  redis-config: |
    maxmemory-policy volatile-lru
    maxmemory 2560gb
    maxmemory-samples 5
    timeout 0
    tcp-keepalive 300
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    hz 40
    activerehashing yes
    aof-rewrite-incremental-fsync yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    save 900 1
    save 300 10
    save 60 10000
    repl-ping-slave-period 10
    repl-timeout 60
    min-slaves-to-write 3
    min-slaves-max-lag 10
    min-replicas-to-write 3
    min-replicas-max-lag 10
EOF
```

Apply the ConfigMap created above, along with a Redis pod manifest:

```bash
kubectl apply -f example-redis-config.yaml
```

Download basic redis-pod.yaml

```bash
wget https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

Probably you may need to adjust at least paths on filesystem:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/storage/pool-1/redis/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /storage/pool-1/redis/redis-master-data
      name: data
    - mountPath: /storage/pool-1/redis/redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: redis-config
        items:
        - key: redis-config
          path: redis.conf

```

then apply deployment:

```bash
kubectl apply -f redis-pod.yaml
```

Examine the created objects:

```bash
kubectl get pod/redis configmap/redis-config
```

You should see the following output:

```bash
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          93s

NAME                     DATA   AGE
configmap/redis-config   1      111s
```

```bash
kubectl describe configmap/redis-config
```

```bash
Name:         redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory-policy volatile-lru
maxmemory 2560gb
maxmemory-samples 5
timeout 0
tcp-keepalive 300
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
hz 40
activerehashing yes
aof-rewrite-incremental-fsync yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
save 900 1
save 300 10
save 60 10000
repl-ping-slave-period 10
repl-timeout 60
min-slaves-to-write 3
min-slaves-max-lag 10
min-replicas-to-write 3
min-replicas-max-lag 10


BinaryData
====

Events:  <none>
```

Every time you change ConfigMap, you have to recreate pod

```bash
kubectl delete pod redis
kubectl apply -f redis-pod.yaml
```


```bash
kubectl exec -it redis -- redis-cli
```

## Uninstall redis

```bash
kubectl delete pod/redis configmap/redis-config
```