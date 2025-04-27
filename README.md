# k3d-obs
small k8s cluster to be used for observability demos.

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
# node mixin 
```shell
❯ cd mixin/node
❯ make node-rules.yaml node-alert.yaml
❯ make dashboards_out
jsonnet -J vendor -m dashboards_out dashboards.jsonnet
dashboards_out/node-cluster-rsrc-use.json
dashboards_out/node-rsrc-use.json
dashboards_out/nodes-darwin.json
dashboards_out/nodes.json

❯ promtool check rules node_rules.yaml
Checking node_rules.yaml
  SUCCESS: 11 rules found

❯ kubectl create configmap -n monitoring node-rules --from-file=node_rules.yaml --dry-run=client -o yaml > node_rules_configmap.yaml
```



