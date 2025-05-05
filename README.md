# k3d-obs
small k8s cluster to be used for observability demos.  To use this for yourself, fork this repo 
and use your fork to follow the steps below.

Create the cluster
```shell
❯ ulimit -n 65536

❯ k3d cluster create --config=k3d.yaml
INFO[0000] Using config file k3d.yaml (k3d.io/v1alpha4#simple)
INFO[0000] portmapping '8081:80' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] portmapping '443:443' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] Prep: Network                                
INFO[0000] Created network 'k3d-obs'                    
INFO[0000] Created image volume k3d-obs-images          
INFO[0000] Starting new tools node...                   
INFO[0000] Starting Node 'k3d-obs-tools'                
INFO[0001] Creating node 'k3d-obs-server-0'             
INFO[0001] Creating node 'k3d-obs-agent-0'              
INFO[0001] Creating node 'k3d-obs-agent-1'              
INFO[0001] Creating node 'k3d-obs-agent-2'              
INFO[0001] Creating LoadBalancer 'k3d-obs-serverlb'     
INFO[0001] Using the k3d-tools node to gather environment information
INFO[0001] HostIP: using network gateway 172.25.0.1 address
INFO[0001] Starting cluster 'obs'                       
INFO[0001] Starting servers...                          
INFO[0001] Starting Node 'k3d-obs-server-0'             
INFO[0006] Starting agents...                           
INFO[0007] Starting Node 'k3d-obs-agent-0'              
INFO[0007] Starting Node 'k3d-obs-agent-2'              
INFO[0007] Starting Node 'k3d-obs-agent-1'              
INFO[0017] Starting helpers...                          
INFO[0017] Starting Node 'k3d-obs-serverlb'             
INFO[0023] Injecting records for hostAliases (incl. host.k3d.internal) and for 5 network members into CoreDNS configmap...
INFO[0026] Cluster 'obs' created successfully!          
INFO[0026] You can now use it like this:                
kubectl cluster-info
```

# Boostrap (this'll take a few minutes)
Set your GITHUB_USER variable, then do the following
```shell
flux --version
flux version  2.5.1

flux bootstrap github \
--owner=$GITHUB_USER \
--repository=k3d-obs \
--branch=main \
--path=./ops \
--personal
```
While it bootstraps, you can have a look at it's process by checking
```
> kg kustomization -A
NAMESPACE     NAME                    AGE   READY   STATUS
flux-system   akhq                    10m   False   dependency 'flux-system/kafka' is not ready
flux-system   flux-system             10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   k6-trace-test           10m   False   dependency 'flux-system/otel-ingester' is not ready
flux-system   kafka                   10m   False   dependency 'flux-system/kube-prometheus-stack' is not ready
flux-system   kafka-operator          10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   kube-prometheus-stack   10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   loki                    10m   False   dependency 'flux-system/kube-prometheus-stack' is not ready
flux-system   minio-operator          10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   operators               10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   otel-distributor        10m   False   dependency 'flux-system/kafka' is not ready
flux-system   otel-ingester           10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   otel-processor          10m   False   dependency 'flux-system/tempo' is not ready
flux-system   releases                10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   repos                   10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   supporting-infra        10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   supporting-infra-ns     10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
flux-system   tempo                   10m   True    Applied revision: main@sha1:7168a1e8d53baf556b904098d7c479619bd39444
```
and 
```
> kg helmrelease -A
NAMESPACE     NAME                  AGE     READY     STATUS
flux-system   akhq                  118s    True      Helm install succeeded for release queues/queues-akhq.v1 with chart akhq@0.25.1
flux-system   kafka-operator        13m     True      Helm install succeeded for release queues/queues-kafka-operator.v1 with chart strimzi-kafka-operator@0.40.0
flux-system   loki-stack            2m28s   True      Helm install succeeded for release logs/logs-loki-stack.v1 with chart loki-stack@2.10.2
flux-system   minio-operator        13m     True      Helm install succeeded for release storage/storage-minio-operator.v1 with chart minio-operator@0.1.7
flux-system   otel-distributor      117s    True      Helm install succeeded for release trace-pipeline/trace-pipeline-otel-distributor.v1 with chart opentelemetry-collector@0.62.3
flux-system   otel-ingester         2m57s   Unknown   Running 'install' action with timeout of 5m0s
flux-system   otel-processor        2m27s   True      Helm install succeeded for release trace-pipeline/trace-pipeline-otel-processor.v1 with chart opentelemetry-collector@0.62.3
flux-system   prometheus-operator   2m58s   True      Helm install succeeded for release metrics/metrics-prometheus-operator.v1 with chart kube-prometheus-stack@71.2.0
flux-system   tempo                 2m57s   True      Helm install succeeded for release traces/traces-tempo.v1 with chart tempo@1.21.0
```

The dependencies have been set so that things should come up after a while 
with a minimum of fuss.

