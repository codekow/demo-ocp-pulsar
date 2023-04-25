# Pulsar Info

## Command Dump

### Links

- https://github.com/datastax/pulsar
- https://github.com/datastax/pulsar-helm-chart
- https://github.com/datastax/pulsar-admin-console

```
cat << YAML > scratch/dev-values.yaml
enableAntiAffinity: false
enableTls: false
enableTokenAuth: false
restartOnConfigMapChange:
  enabled: true
extra:
  function: true
  burnell: true
  burnellLogCollector: true
  pulsarHeartbeat: true
  pulsarAdminConsole: true

zookeeper:
  replicaCount: 1
  resources:
    requests:
      memory: 300Mi
      cpu: 0.3
  configData:
    PULSAR_MEM: "-Xms300m -Xmx300m -Djute.maxbuffer=10485760 -XX:+ExitOnOutOfMemoryError"

bookkeeper:
  replicaCount: 1
  resources:
    requests:
      memory: 512Mi
      cpu: 0.3
  configData:
    BOOKIE_MEM: "-Xms312m -Xmx312m -XX:MaxDirectMemorySize=200m -XX:+ExitOnOutOfMemoryError"

broker:
  component: broker
  replicaCount: 1
  ledger:
    defaultEnsembleSize: 1
    defaultAckQuorum: 1
    defaultWriteQuorum: 1
  resources:
    requests:
      memory: 600Mi
      cpu: 0.3
  configData:
    PULSAR_MEM: "-Xms400m -Xmx400m -XX:MaxDirectMemorySize=200m -XX:+ExitOnOutOfMemoryError"

autoRecovery:
  resources:
    requests:
      memory: 300Mi
      cpu: 0.3

function:
  replicaCount: 1
  functionReplicaCount: 1
  resources:
    requests:
      memory: 512Mi
      cpu: 0.3
  configData:
    PULSAR_MEM: "-Xms312m -Xmx312m -XX:MaxDirectMemorySize=200m -XX:+ExitOnOutOfMemoryError"

proxy:
  replicaCount: 1
  resources:
    requests:
      memory: 512Mi
      cpu: 0.3
  wsResources:
    requests:
      memory: 512Mi
      cpu: 0.3
  configData:
    PULSAR_MEM: "-Xms400m -Xmx400m -XX:MaxDirectMemorySize=112m"
  autoPortAssign:
    enablePlainTextWithTLS: true
  service:
    autoPortAssign:
      enabled: true

grafanaDashboards:
  enabled: true

pulsarAdminConsole:
  replicaCount: 1

# kube-prometheus-stack:
#  enabled: true
#  prometheusOperator:
#    enabled: true
#  grafana:
#    enabled: true
#    adminPassword: verysecretmuch
YAML
```

```
oc new-project pulsar
helm repo add datastax-pulsar https://datastax.github.io/pulsar-helm-chart
helm repo update
helm install pulsar datastax-pulsar/pulsar -n pulsar

oc patch statefulset pulsar-bookkeeper --type=json \
  -p '[{"op": "remove", "path": "/spec/template/spec/securityContext/fsGroup"}]'

oc patch statefulset pulsar-function --type=json \
  -p '[{"op": "remove", "path": "/spec/template/spec/securityContext/fsGroup"}]'

oc patch statefulset pulsar-zookeeper --type=json \
  -p '[{"op": "remove", "path": "/spec/template/spec/securityContext/fsGroup"}]'

```

```
# helm uninstall pulsar -n pulsar
# helm upgrade pulsar datastax-pulsar/pulsar -n pulsar -f scratch/dev-values.yaml

helm install pulsar datastax-pulsar/pulsar -n pulsar -f scratch/dev-values.yaml
```

### Errors

Command: `helm install pulsar datastax-pulsar/pulsar -n pulsar -f scratch/dev-values.yaml`

```
# kube-prometheus-stack:
#  enabled: true
#  prometheusOperator:
#    enabled: true
#  grafana:
#    enabled: true
#    adminPassword: verysecretmuch
Error: UPGRADE FAILED: error validating "": error validating data: ValidationError(Prometheus.spec): unknown field "hostNetwork" in com.coreos.monitoring.v1.Prometheus.spec
```

Deployment / pulsar-adminconsole

```
npm ERR! code EACCES
npm ERR! syscall mkdir
npm ERR! path /home/appuser/.npm/_cacache
npm ERR! errno -13
npm ERR! 
npm ERR! Your cache folder contains root-owned files, due to a bug in
npm ERR! previous versions of npm which has since been addressed.
npm ERR! 
npm ERR! To permanently fix this problem, please run:
npm ERR!   sudo chown -R 1000850000:0 "/home/appuser/.npm"

npm ERR! Log files were not written due to an error writing to the directory: /home/appuser/.npm/_logs
npm ERR! You can rerun the command with `--loglevel=verbose` to see the logs in your terminal
```
