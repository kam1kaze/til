## How to find out of memory (OOM) pods


Find OOMkilled pods
```
kubectl get pods --all-namespaces -o json \
  | jq -c '.items[] | select(.status.containerStatuses[].lastState.terminated.reason=="OOMKilled") | [.metadata.name, (.status.containerStatuses[].lastState.terminated | select(.reason=="OOMKilled") | .finishedAt)]'
```

Find the node name where OOM occurs 
```
kubectl get events --all-namespaces -o json \
  | jq -c '.items[]| select(.reason=="OOMKilling")| [ .lastTimestamp, .metadata.namespace, .involvedObject.name, ( .message | split("\n")[1] )]' \
  | sort
```
