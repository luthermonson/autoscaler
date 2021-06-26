Rancher kubernetes autoscaler provides you out-of-the box facility to scale your kubernetes cluster.

Intallation Method:

1. Create a sperate worker node-pool. Let it be "scaler" for this guide. You are free to choose any name/identifier of your choice.
    - This node-pool will contain the nodes that will be autoscaled.
2. Create a secret using the following snippet. Make sure to update url , access-token , secret and cluster-id as per your rancher cluster.

```
apiVersion: v1
kind: Secret
metadata:
  name: cluster-autoscaler-cloud-config
  namespace: kube-system
type: Opaque
stringData:
  cloud-config: |
    [Global]
    url=https://rancher.com/v3
    access=token-tttttt
    secret=secretsecretsecretsecretsecret
    cluster-id=c-qwerty
  autoscaler_node_arg: "2:6:c-abcdef:np-abcde"
```
NOTE:
Take a note to `autoscaler_node_arg` . Here the value follows the following nomenclature.
<MIN_NODE_COUNT_SCALING_NODE_POOL>:<MAX_NODE_COUNT_SCALING_NODE_POOL>:<CLUSTER_ID>:<NODE_POOL_ID>

3. Once you have applied the above deatils save the above file as `cluster-autoscaler-cloud-config.yaml`. Now apply the apply the secret to the kubernetes cluster either by importing the yaml from rancher portal or adding the secret manually from portal or use the kubectl command -
`kubectl apply -f cluster-autoscaler-cloud-config.yaml`
NOTE: Since its auto-scaler you can add it to either kube-system namespace or any other namespace that you desire. Here we will use `kube-system` namespace.

4. Add catalog https://github.com/kubernetes/autoscaler to rancher
   This will add the helm chart of kubernetes auto-scaler to the rancher catalog.

5. Now we have to install the autoscaler adding the following values: 
    autoscalingGroups[0].maxSize: <MAX_LIMIT_OF_AUTOSCALED_NODES>
    autoscalingGroups[0].minSize: <MIN_LIMIT_OF_AUTOSCALED_NODES>
    autoscalingGroups[0].name: <c-abc:np-xyz>   #--> this will depend on your cluster c-abc = cluster id and np-xyz= node pool id
    cloudProvider: rancher
    extraArgs.cloud-config: /config/cloud-config
    extraVolumeSecrets.cluster-autoscaler-cloud-config.mountPath: /config
    extraVolumeSecrets.cluster-autoscaler-cloud-config.name: cluster-autoscaler-cloud-config
    image.repository: luthermonson/autoscaler
    image.tag: rancher

