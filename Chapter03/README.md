## 파드 이해하기
- 파드의 모든 컨테이너는 동일한 네트워크 네임 스페이스와 동일한 UTS 네임스페이스(UNIX Timesharing System Namespace, 호스트 이름을 네임스페이스별로 격리)안에서 실행
- 그래서 모든 컨테이너는 같은 호스트 이름과 네트워크 인터페이스를 공유
- 또한 파드의 모든 컨테이너는 동일한 IPC 네임스페이스 아래에서 실행되어, IPC를 통해 서로 통신 가능
- 그러나 파일 시스템에 대해서는 조금 다름
    - 대부분의 컨테이너 파일 시스템은 컨테이너 이미지에서 나오기 때문에, 기본적으로 파일시스템은 다른 컨테이너와 완전히 분리
    - 그러나 쿠버네티스 볼륨 개념을 이용해서 컨테이너가 파일 디렉터리를 공유하는 것이 가능함.(추후 공부)
    

## 컨테이너가 동일한 IP와 포트 공간을 공유하는 방법
- 파드 안의 컨테이너가 동일한 네트워크 네임스페이스에서 실행된다는 것을 강조하고 싶다.
- 그렇기에 동일한 IP 주소와 포트 공간을 공유한다.
- 이 말은 동일한 파드 안에서 실행중인 프로세스가 같은 포트 번호를 사용하지 않도록 주의해야 한다는 걸 의미
- 이는 물론 동일한 파드일 때만 해당함
    - 다른 파드에 있는 컨테이너는 서로 다른 포트 공간을 갖음
- 파드 안에 있는 모든 컨테이너는 동일한 루프백 네트워크 인터페이스를 갖기 때문에 컨테이너들이 로컬 호스트를 통해 서로 통신할 수 있다.


## 플랫 네트워크?

- 쿠버네티스 클러스터의 모든 파드는 하나의 플랫한 공유 네트워크 주소 공간에 상주
- 따라서 모든 파드는 다른 파드의 IP 주소를 사용해 접근하는 것이 가능
- 둘 사이에는 어떠한 NAT(Network Address Translation)도 존재하지 않음
- 그 말인 즉, 두 파드가 서로 네트워크 패킷을 보내면, 상대방의 실제 IP 주소를 패킷 안에 있는 출발지 IP 주소에서 찾을 수 있다는 것

- 결과적으로 말하고 싶은 건 파드 사이에서 통신은 항상 단순하다는 것.
- 두 파드가 동일 혹은 서로 다른 워커 노드에 있는지는 중요하지 않고
- 두 경우 모두 파드 안에 있는 컨테이너는 NAT 없는 플랫 네트워크를 통해 서로 통신하는 것이 가능
- 각 파드는 고유 IP를 가지고, 다른 파드에서 이 네트워크를 통해 접속 가능하다는 것

파드는 논리적인 호스트 -> VM과 매우 유사하게 동작함


## 다계층 애플리케이션을 여러 파드로 분할

웹서버와 데이터베이스가 정말 같은 머신에서 실행되어야 하는가? -> 아니오
만약 같은 파드에 프론트엔드, 백엔드를 두자. 둘은 항상 같은 노드에서 실행됨
만약 워커 노드가 여러개라면 컴퓨팅 리소스(CPU, Memory)를 활영하지 않고 그냥 버림
여러개로 쪼개면 각각 다른 노드에 스케줄링해서 인프라스트럭쳐의 활용도를 향상시킬 수 있음.

- 한 곳에 때려넣으면 스케일링 문제가 있음
- 스케일링의 기본 단위는 파드임!!
- 쿠버네티스는 개별 컨테이너를 수평으로 확장할 수 없음
- 대신 전체 파드를 수평으로 확장함
- 만약에 FE, BE 컨테이너로 구성된 파드를 2개로 늘린다? -> 결국 FE 컨테이너 2개, BE 컨테이너 2개가 됨
- 일반적으로 FE 구성요소는 백엔드와 완전히 다른 스케일링 요구 사항을 갖고 있음 -> (개별적으로 확장함)
- 데이터베이스와 같은 백엔드는(상태를 갖지 않는) 프론트엔드 웹서버에 비해 확장하기가 훨씬 어려움
- 컨테이너를 개별적으로 스케일링하는 것이 필요하다? -> 별도 파드에 배포해야 한다.

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
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master ✚  minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:62349/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...


^C
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  kubectl get jobs
NAME        COMPLETIONS   DURATION   AGE
batch-job   1/1           2m6s       12d
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  kubectl delete job batch-job
job.batch "batch-job" deleted
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  minikube dashboard          
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:62408/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
^C
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  
 ✘ user  ~/Documents/github/kubenetes-in-action/Chapter02   master ±✚  git add .
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master ✚  git commit -m "Chapter02"
[master 5987309] Chapter02
 4 files changed, 66 insertions(+), 1 deletion(-)
 create mode 100644 Chapter02/README.md
 rename Chapter02/{kubia.yaml => dolphago.yaml} (100%)
 rename Chapter02/{kubia => dolphago}/Dockerfile (100%)
 rename Chapter02/{kubia => dolphago}/app.js (87%)
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master  git push origin master  
오브젝트 나열하는 중: 9, 완료.
오브젝트 개수 세는 중: 100% (9/9), 완료.
Delta compression using up to 12 threads
오브젝트 압축하는 중: 100% (7/7), 완료.
오브젝트 쓰는 중: 100% (7/7), 1.61 KiB | 1.61 MiB/s, 완료.
Total 7 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/adamdoha/kubenetes-in-action.git
   127ac15..5987309  master -> master
 user  ~/Documents/github/kubenetes-in-action/Chapter02   master  cd ..
 user  ~/Documents/github/kubenetes-in-action   master  cd Chapter03
 user  ~/Documents/github/kubenetes-in-action/Chapter03   master  ls
custom-namespace.yaml              kubia-manual-custom-namespace.yaml kubia-manual.yaml
kubia-gpu.yaml                     kubia-manual-with-labels.yaml
 user  ~/Documents/github/kubenetes-in-action/Chapter03   master ✚  ls
README.md                          kubia-manual-custom-namespace.yaml
custom-namespace.yaml              kubia-manual-with-labels.yaml
kubia-gpu.yaml                     kubia-manual.yaml
 user  ~/Documents/github/kubenetes-in-action/Chapter03   master ±✚  kubectl get po dolphago -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-03-02T22:33:06Z"
  labels:
    run: dolphago
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:run: {}
      f:spec:
        f:containers:
          k:{"name":"dolphago"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:ports:
              .: {}
              k:{"containerPort":8080,"protocol":"TCP"}:
                .: {}
                f:containerPort: {}
                f:protocol: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kubectl-run
    operation: Update
    time: "2021-03-02T22:33:06Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"172.17.0.5"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2021-03-02T22:33:13Z"
  name: dolphago
  namespace: default
  resourceVersion: "586661"
  uid: f76055de-5f8a-48a4-9259-5310c5716106
spec:
  containers:
  - image: adamdoha/dolphago
    imagePullPolicy: Always
    name: dolphago
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-9q5dr
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-9q5dr
    secret:
      defaultMode: 420
      secretName: default-token-9q5dr
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-03-02T22:33:06Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2021-03-02T22:33:13Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2021-03-02T22:33:13Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-03-02T22:33:06Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://88895b9095fff182a8b165bfe2aa1cee18059f46f0eca1f6b8038d8d61af3252
    image: adamdoha/dolphago:latest
    imageID: docker-pullable://adamdoha/dolphago@sha256:1af55e02eb8aa35913971154d25982563476790ce6ed254643e9e29589b8f7eb
    lastState: {}
    name: dolphago
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2021-03-02T22:33:12Z"
  hostIP: 192.168.64.2
  phase: Running
  podIP: 172.17.0.5
  podIPs:
  - ip: 172.17.0.5
  qosClass: BestEffort
  startTime: "2021-03-02T22:33:06Z"

```

## 파드를 정의하는 주요 부분
- Metadata: 이름, 네임스페이스, 레이블 및 파드에 관한 기타 정보 포함
- Spec : 파드 컨테이너, 볼륨, 기타 데이터 등 파드 자체에 관한 실제 명세
- Status : 파드 상태, 각 컨테이너 설명과 상태, 파드 내부 IP, 기타 기본 정보 등 현재 실행 중인 파드에 관한 현재 정보 포함


### kubectl explain을 이용해 사용가능한 API 오브젝트 필드 찾
```
$ kubectl explain pods
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

### kubectl create 로 파드 만들기
```
kubectl create -f dolphago-manual.yaml
```

여기서 사용한 kubectl create -f 명령은 yaml 또는 json 파일로 파드 뿐만 아니라 리소스를 만드는데 사용한다.

### 파드 전체 정의 보기
파드 정의를 yaml로 보고싶다면
```
kubectl get po dolphago-manual -o yaml
```
파드 정의를 json으로 보고 싶다
```
kubectl get po dolphago-manual -o json
```

### 파드 목록에서 새로 생성된 파드 보기

```
kubectl get pods
```

### kubectl logs 를 이용해서 파드 로그 가져오기

```
kubectl logs dolphago-manual
DolphaGo server starting...
```


### 포트 포워딩
```
$ kubectl port-forward dolphago-manual 8888:8080
Forwarding from 127.0.0.1:8888 -> 8080
Forwarding from [::1]:8888 -> 8080
```
위와 같이 하면 포트포워딩이 실행되어 로컬 포트로 파드에 연결할 수가 있게 된다.
즉, localhost:8888로 파드에 접근할 수 있다.
```
$ curl localhost:8888

You've hit dolphago-manual
```

신기하네 포트포워딩 ㅎㅎㅎㅎ(130p 참조)

### Kubernetes에서 Annotation이 뭐죠?

파드 및 다른 오브젝트들은 Label 외에 annotations를 가질 수 있습니다.

Annotation은 키-값 쌍으로 레이블과 거의 비슷하지만, 식별 정보를 갖지 않습니다.

레이블은 오브젝트를 묶는 데 사용할 수 있지만, 어노테이션은 그렇게 할 수 없습니다.

(이전에 Label Selector를 통해 오브젝트를 입맛에 맞게 선택했었지만, Annotation Selector는 없습니다.)

### 그럼 레이블을 쓰면 되는데, 어노테이션을 왜 써요?

어노테이션은 훨씬 더 많은 정보를 보유할 수 있습니다.

이 정보들은 주로 tools들에서 사용되는 정보들입니다.

특정 Annotation은 Kubernetes에 의해서 자동으로 Object에 추가가 되지만,

나머지 어노테이션은 사용자가 직접 추가합니다.

### 어노테이션 사용의 예

쿠버네티스에 새로운 기능을 추가할 때 흔히 사용합니다.

예를 들면, 새로운 기능의 알파/베타 버전은 API 오브젝트에 새로운 필드를 바로 도입하지 않습니다.

필드 대신 어노테이션을 사용하고, 필요한 API 변경이 명확해지고 쿠버네티스 개발자가 이에 동의하면 새로운 필드가 도입되는 방식입니다. 그리고 관련 어노테이션은 사라지게 됩니다.

또한 파드나 다른 API 오브젝트에 설명을 추가해놓으면,  클러스터를 사용하는 모든 사람이 개별 오브젝트에 관한 정보를 신속하게 찾아볼 수 있습니다.

### 기존에 만든 파드의 어노테이션 확인해보기

파드 생성 시, 쿠버네티스가 자동으로 만들어준 annotation을 확인해보겠습니다.

어노테이션을 보기 위해서는 describe 명령이나, yaml 전체 내용을 요청해야 합니다.

`kubectl get po {파드 이름} -o yaml`
$ kubectl get po kubia-manual -o yaml
apiVersion: v1
kind: Pod
metadata:
annotations:
kubernetes.io/psp: default
creationTimestamp: "2021-04-11T16:13:42Z"
labels:
app: dolphago
...
kubernetes.io/psp 어노테이션이 default로 지정되어 있다. 라고 되어있네요.

annotations:
kubernetes.io/psp: default
여기서 psp는 pod security policy를 의미하는데요. 이 정책이 default라고 설명되어 있는 것을 확인할 수 있습니다.

이런 데이터를 레이블에 굳이 넣고 싶지는 않겠죠? 어노테이션에 적절히 활용할 수 있겠습니다.

레이블에는 짧은 데이터를, 어노테이션에서는 상대적으로 큰 데이터를 넣을 수 있습니다.(256KB)

### 어노테이션 추가/수정/삭제

- 레이블을 추가/수정했던 것과 마찬가지로, 파드를 생성할 때나 이미 만들어진 파드에 어노테이션을 추가/수정도 가능합니다.

- 레이블에서 kubectl label ~ 로 레이블링을 했던 것처럼, 어노테이션에서도 kubectl annotate 명령을 사용합니다.

저는 기존에 만들어둔 파드 kubia-manual 이라는 파드가 있는데요.
여기에 어노테이션이 kubernetes.io/psp: default 밖에 없는 상황입니다.

어노테이션을 추가하는 명령어는 다음과 같습니다.

kubectl annotate pod {파드이름} {키}={값}
파드에 어노테이션이 추가가 되었는지 확인하려면 다음 명령어로 확인합니다. (아니면 -o yaml이나 -o json으로 출력하시면 됩니다.)
`kubectl describe pod {파드 이름}`

어노테이션을 수정하는 것도 레이블링 방식과 동일합니다.
변경할 키=값 끝에 --overwrite 옵션을 추가해주면 됩니다.

`kubectl annotate pod {파드이름} {키}={값} --overwrite`
어노테이션을 삭제하는 명령어는 다음과 같습니다.

`kubectl annotate pod {파드이름} {키}-`
삭제하려는 {키} 끝에  -를 붙여주면 됩니다.
