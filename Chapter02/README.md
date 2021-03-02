

```
kubectl describe po <pod name>
```

```
kubectl describe po dolphago
Name:         dolphago
Namespace:    default
Priority:     0
Node:         minikube/192.168.64.2
Start Time:   Wed, 03 Mar 2021 07:33:06 +0900
Labels:       run=dolphago
Annotations:  <none>
Status:       Running
IP:           172.17.0.5
IPs:
  IP:  172.17.0.5
Containers:
  dolphago:
    Container ID:   docker://88895b9095fff182a8b165bfe2aa1cee18059f46f0eca1f6b8038d8d61af3252
    Image:          adamdoha/dolphago
    Image ID:       docker-pullable://adamdoha/dolphago@sha256:1af55e02eb8aa35913971154d25982563476790ce6ed254643e9e29589b8f7eb
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 03 Mar 2021 07:33:12 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9q5dr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-9q5dr:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9q5dr
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  9m59s  default-scheduler  Successfully assigned default/dolphago to minikube
  Normal  Pulling    9m58s  kubelet            Pulling image "adamdoha/dolphago"
  Normal  Pulled     9m53s  kubelet            Successfully pulled image "adamdoha/dolphago" in 5.496932487s
  Normal  Created    9m53s  kubelet            Created container dolphago
  Normal  Started    9m53s  kubelet            Started container dolphago

```


minikube의 dashboard 확인은
```
minikube dashboard
```

