# NodeSelector
this mean pod create which node. NodeSelector option deployment create pod in this label node. For example:
```
# oc get node --show-labels
NAME                   STATUS   ROLES    AGE    VERSION                LABELS
ocp-w-1.lab.ocp.lan    Ready    worker   336d   v1.20.7+bbbc079-1330   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,cluster.ocs.openshift.io/openshift-storage=,kubernetes.io/arch=amd64,kubernetes.io/hostname=ocp-w-1.lab.ocp.lan,kubernetes.io/os=linux,node-role.kubernetes.io/worker=,node.openshift.io/os_id=fedora
ocp-w-2.lab.ocp.lan    Ready    worker   336d   v1.20.7+bbbc079-1330   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,cluster.ocs.openshift.io/openshift-storage=,kubernetes.io/arch=amd64,kubernetes.io/hostname=ocp-w-2.lab.ocp.lan,kubernetes.io/os=linux,node-role.kubernetes.io/worker=,node.openshift.io/os_id=fedora,worker=node02
```
Above this command we look 'ocp-w-2.lab.ocp.lan' node label is 'worker=node02' , in last section. But 'ocp-w-1.lab.ocp.lan' node default labels. When i create deployment all containers created only 'ocp-w-1.lab.ocp.lan' node. When i change deployment 'nodeSelector' option to 'worker=node02' this label, this time pods termitaing and created only 'ocp-w-2.lab.ocp.lan' node. Below i edit deployment 'nodeSelector' option
```
# oc create deployment nginx --image=bitnami/nginx
# oc scale --replicas 8 deployment nginx
# oc get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE                  NOMINATED NODE   READINESS GATES
nginx-796f99c467-4t52m   1/1     Running   0          63s    10.128.3.116   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-4xhp4   1/1     Running   0          63s    10.128.3.113   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-66hqk   1/1     Running   0          76s    10.128.3.112   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-74m9s   1/1     Running   0          76s    10.128.3.111   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-9svmj   1/1     Running   0          76s    10.128.3.110   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-j4wm2   1/1     Running   0          111s   10.128.3.109   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-jmwcl   1/1     Running   0          63s    10.128.3.115   ocp-w-1.lab.ocp.lan   <none>           <none>
nginx-796f99c467-pqp7b   1/1     Running   0          63s    10.128.3.114   ocp-w-1.lab.ocp.lan   <none>           <none>
# oc edit deployment nginx
    spec:
      containers:
      - image: bitnami/nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:  # this section add 'nodeSelector'
        worker: node02
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

# oc get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP             NODE                  NOMINATED NODE   READINESS GATES
nginx-cdd6c6f4b-2pnd6   1/1     Running   0          29m   10.131.1.137   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-f4hp6   1/1     Running   0          29m   10.131.1.138   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-j7ssk   1/1     Running   0          28m   10.131.1.144   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-kppcv   1/1     Running   0          29m   10.131.1.141   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-php2m   1/1     Running   0          29m   10.131.1.142   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-qj2qn   1/1     Running   0          29m   10.131.1.143   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-xg7h2   1/1     Running   0          29m   10.131.1.139   ocp-w-2.lab.ocp.lan   <none>           <none>
nginx-cdd6c6f4b-xv925   1/1     Running   0          29m   10.131.1.140   ocp-w-2.lab.ocp.lan   <none>           <none>
```
Above i edit deployment  and add line 'nodeSelector' section. And after 1 minute all pods running to 'ocp-w-2.lab.ocp.lan' node.

# Horizontal Auto Scaling Pod
When pod running very heavy, pods autocreating more with horizontal autoscaling:
```
# oc  create deployment nginx --image=bitnami/nginx
# oc get pod
NAME                    READY   STATUS    RESTARTS   AGE
nginx-c79ccbdc7-6vxfl   1/1     Running   0          4m22s
# oc set resources deploy nginx --requests cpu=5m,memory=100Mi --limits cpu=5m,memory=500Mi
# oc autoscale deploy nginx --min 1 --max 10 --cpu-percent 5
```
Above i create 'nginx' deployment and add resource. End of line i create horizontal autoscale. '--min 1' -mean to when  i create this , thic deployment scale minimun 1 pods; '--max 10'  -mean when i create this, this nginx deployment scale maximum 10 pods. Current 1 pods working. Below command i stress this deployment:
```
# while true; do wget -q -O- nginx-myproject.apps.lab.ocp.lan; done
```
And about 10 minutes pods running stress. And below result. Pods scale to 10 pods:
```
# oc get pod
NAME                    READY   STATUS    RESTARTS   AGE
nginx-c79ccbdc7-6vxfl   1/1     Running   0          4m22s
nginx-c79ccbdc7-72dch   1/1     Running   0          4m38s
nginx-c79ccbdc7-89xxb   1/1     Running   0          4m22s
nginx-c79ccbdc7-992tb   1/1     Running   0          4m38s
nginx-c79ccbdc7-dcdpw   1/1     Running   0          10m
nginx-c79ccbdc7-dz6ng   1/1     Running   0          4m53s
nginx-c79ccbdc7-hhdwl   1/1     Running   0          4m38s
nginx-c79ccbdc7-l7z2q   1/1     Running   0          4m53s
nginx-c79ccbdc7-nz8g9   1/1     Running   0          4m37s
nginx-c79ccbdc7-vzwmv   1/1     Running   0          4m53s
```
Below the command i look 'TARGETS' section this deployment load '220%' cpu:
```
# oc get hpa
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   220%/5%   1         10        10         3m53s
```
When stress stopping the pods scale to 1 pod.

# Pod Resource Limitations
At below i create deployment and set resource limits:
```
# oc create deployment mydeployment --image=bitnami/nginx
# oc set resources deploy mydeployment --requests cpu=10m,memory=1Mi --limits cpu=20m,memory=5Mi
# oc get pod
NAME                           READY   STATUS              RESTARTS   AGE
mydeployment-694bf8776-2kg4t   1/1     Running             0          32s
mydeployment-864d55b6-5fbdg    0/1     ContainerCreating   0          10s
# oc describe pod mydeployment-864d55b6-5fbdg
Name:           mydeployment-864d55b6-5fbdg
Namespace:      myproject
Priority:       0
Node:           ocp-w-1.lab.ocp.lan/192.168.52.217
Start Time:     Tue, 31 May 2022 11:01:12 +0400
Labels:         app=mydeployment
                pod-template-hash=864d55b6
Annotations:    openshift.io/scc: restricted
Status:         Pending
IP:
IPs:            <none>
Controlled By:  ReplicaSet/mydeployment-864d55b6
Containers:
  nginx:
    Container ID:
    Image:          bitnami/nginx
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     20m
      memory:  5Mi
    Requests:
      cpu:        10m
      memory:     1Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-t94nw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-t94nw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-t94nw
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age               From               Message
  ----     ------                  ----              ----               -------
  Normal   Scheduled               15s               default-scheduler  Successfully assigned myproject/mydeployment-864d55b6-5fbdg to ocp-w-1.lab.ocp.lan
  Warning  FailedCreatePodSandBox  3s (x2 over 15s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = pod set memory limit 5242880 too low; should be at least 12582912
```
At Above when i set resource 'mydeployment-864d55b6-5fbdg' created. This pod is problem because applied resources are few. We look warning this pod 'Failed to create pod sandbox: rpc error: code = Unknown desc = pod set memory limit 5242880 too low; should be at least 12582912' . Thats why  i again set resource:
```
# oc set resources deploy mydeployment --requests cpu=10m,memory=50Mi --limits cpu=20m,memory=100Mi
# oc get pod
NAME                            READY   STATUS    RESTARTS   AGE
mydeployment-674b49d4f6-n8cqh   1/1     Running   0          23s
```
At that moment container running. Because this resources are true.

# Specifying Quota Range
This task i create clusterquota and join to namespaces. Only env=testing namespaces work this quota:
```
# oc create clusterquota testing  --project-label-selector env=testing --hard pods=5,services=2
# oc describe clusterquota testing -A
Name:           testing
Created:        8 minutes ago
Labels:         <none>
Annotations:    <none>
Namespace Selector: ["myproject"]
Label Selector: env=testing
AnnotationSelector: map[]
Resource        Used    Hard
--------        ----    ----
pods            5       5
services        0       2
# oc create deploy mydeployment --image=bitnami/nginx  --replicas=6
# oc get pod
NAME                           READY   STATUS    RESTARTS   AGE
mydeployment-694bf8776-89jnz   1/1     Running   0          5m12s
mydeployment-694bf8776-89mm8   1/1     Running   0          5m5s
mydeployment-694bf8776-c9bmp   1/1     Running   0          5m12s
mydeployment-694bf8776-fphds   1/1     Running   0          5m12s
mydeployment-694bf8776-qjbjs   1/1     Running   0          5m12s
# oc describe replicaset.apps/mydeployment-694bf8776
Name:           mydeployment-694bf8776
Namespace:      myproject
Selector:       app=mydeployment,pod-template-hash=694bf8776
Labels:         app=mydeployment
                pod-template-hash=694bf8776
Annotations:    deployment.kubernetes.io/desired-replicas: 6
                deployment.kubernetes.io/max-replicas: 8
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/mydeployment
Replicas:       5 current / 6 desired
Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=mydeployment
           pod-template-hash=694bf8776
  Containers:
   nginx:
    Image:        bitnami/nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  ReplicaFailure   True    FailedCreate
Events:
  Type     Reason            Age                     From                   Message
  ----     ------            ----                    ----                   -------
  Normal   SuccessfulCreate  5m31s                   replicaset-controller  Created pod: mydeployment-694bf8776-fphds
  Normal   SuccessfulCreate  5m31s                   replicaset-controller  Created pod: mydeployment-694bf8776-qjbjs
  Normal   SuccessfulCreate  5m31s                   replicaset-controller  Created pod: mydeployment-694bf8776-c9bmp
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-skffw" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-4jthf" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Normal   SuccessfulCreate  5m31s                   replicaset-controller  Created pod: mydeployment-694bf8776-89jnz
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-cgww9" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-mlddc" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-l2lgn" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-8ddxl" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-7p94s" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m31s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-2sbm4" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Warning  FailedCreate      5m30s                   replicaset-controller  Error creating: pods "mydeployment-694bf8776-s4j74" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
  Normal   SuccessfulCreate  5m24s                   replicaset-controller  Created pod: mydeployment-694bf8776-89mm8
  Warning  FailedCreate      5m17s (x11 over 5m30s)  replicaset-controller  (combined from similar events): Error creating: pods "mydeployment-694bf8776-qjpq6" is forbidden: exceeded quota: testing, requested: pods=1, used: pods=5, limited: pods=5
```
Above we are look clusterquota set max 5pods. We are create deploy replicas=6 but create pod number 5. Thats working.

# Quota
Below i create quota max pod requirement '4':
```
# oc create quota qtest --hard pods=4
```
And then create limits:
```
# cat limits.yml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "project-limits"
spec:
  limits:
    - type: "Container"
      max:
        cpu: "20m"
        memory: "20Mi"
      min:
        cpu: "10m"
        memory: "10Mi"
      default:
        cpu: "20m"
        memory: "20Mi"
# oc apply -f limits.yml
```
I describe where i work project:
```
# oc describe project myproject
Name:           myproject
Created:        2 weeks ago
Labels:         env=testing
Annotations:    openshift.io/description=
                openshift.io/display-name=
                openshift.io/requester=manager
                openshift.io/sa.scc.mcs=s0:c33,c17
                openshift.io/sa.scc.supplemental-groups=1001090000/10000
                openshift.io/sa.scc.uid-range=1001090000/10000
Display Name:   <none>
Description:    <none>
Status:         Active
Node Selector:  <none>
Quota:
        Name:           qtest
        Resource        Used    Hard
        --------        ----    ----
        pods            0       4
Resource limits:
        Name:           project-limits
        Type            Resource        Min     Max     Default Request Default Limit   Max Limit/Request Ratio
        ----            --------        ---     ---     --------------- -------------   -----------------------
        Container       cpu             10m     20m     20m             20m             -
        Container       memory          10Mi    20Mi    20Mi            20Mi            -
```
What i set this is describe above the command. Then i create deployment 6 replicas:
```
# oc create deployment mynginx --image=bitnami/nginx --replicas=6
# oc get pod
NAME                       READY   STATUS    RESTARTS   AGE
mynginx-5f8b4bd48f-npxln   1/1     Running   0          21s
mynginx-5f8b4bd48f-pjq9g   1/1     Running   0          21s
mynginx-5f8b4bd48f-r48sm   1/1     Running   0          21s
mynginx-5f8b4bd48f-w9nd6   1/1     Running   0          21s
```
Above we look we are create 6 replicas pod but 4 pod created. Because i set this project max set '4' pods. Now we test limits qtest. I edit limits yml file and set minimum requirements:
```
# cat limits.yml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "project-limits"
spec:
  limits:
    - type: "Container"
      max:
        cpu: "20m"
        memory: "1Mi"
      min:
        cpu: "10m"
        memory: "1Mi"
      default:
        cpu: "20m"
        memory: "1Mi"
# oc describe project myproject
Name:           myproject
Created:        2 weeks ago
Labels:         env=testing
Annotations:    openshift.io/description=
                openshift.io/display-name=
                openshift.io/requester=manager
                openshift.io/sa.scc.mcs=s0:c33,c17
                openshift.io/sa.scc.supplemental-groups=1001090000/10000
                openshift.io/sa.scc.uid-range=1001090000/10000
Display Name:   <none>
Description:    <none>
Status:         Active
Node Selector:  <none>
Quota:
        Name:           qtest
        Resource        Used    Hard
        --------        ----    ----
        pods            0       4
Resource limits:
        Name:           project-limits
        Type            Resource        Min     Max     Default Request Default Limit   Max Limit/Request Ratio
        ----            --------        ---     ---     --------------- -------------   -----------------------
        Container       memory          1Mi     1Mi     1Mi             1Mi             -
        Container       cpu             10m     20m     20m             20m             -
# oc create deployment mynginx-2 --image=bitnami/nginx --replicas=6
# oc get pod
NAME                         READY   STATUS              RESTARTS   AGE
mynginx-2-548ffd8596-4xm7g   0/1     ContainerCreating   0          3s
mynginx-2-548ffd8596-72nbr   0/1     ContainerCreating   0          3s
mynginx-2-548ffd8596-qgj7n   0/1     ContainerCreating   0          3s
mynginx-2-548ffd8596-tkn4l   0/1     ContainerCreating   0          3s
# oc describe pod mynginx-2-548ffd8596-4xm7g
Name:           mynginx-2-548ffd8596-4xm7g
Namespace:      myproject
Priority:       0
Node:           ocp-w-1.lab.ocp.lan/192.168.52.217
Start Time:     Mon, 06 Jun 2022 11:29:15 +0400
Labels:         app=mynginx-2
                pod-template-hash=548ffd8596
Annotations:    kubernetes.io/limit-ranger: LimitRanger plugin set: cpu, memory request for container nginx; cpu, memory limit for container nginx
                openshift.io/scc: restricted
Status:         Pending
IP:
IPs:            <none>
Controlled By:  ReplicaSet/mynginx-2-548ffd8596
Containers:
  nginx:
    Container ID:
    Image:          bitnami/nginx
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     20m
      memory:  1Mi
    Requests:
      cpu:        20m
      memory:     1Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-t94nw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-t94nw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-t94nw
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age               From               Message
  ----     ------                  ----              ----               -------
  Normal   Scheduled               35s               default-scheduler  Successfully assigned myproject/mynginx-2-548ffd8596-4xm7g to ocp-w-1.lab.ocp.lan
  Warning  FailedCreatePodSandBox  9s (x3 over 35s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = pod set memory limit 1048576 too low; should be at least 12582912
```
Above we recreate deploy and change minimum requirement limits.yml file. And pods not running because log files look 'Failed to create pod sandbox: rpc error: code = Unknown desc = pod set memory limit 1048576 too low; should be at least 12582912'. Minimum requiremnts dont support this pod for running.

# NodeAffinity
This mean it: which pod running which node. This is execute only labels. We want to 'with-node-affinity' pod running only 'ocp-w-2.lab.ocp.lan' node. In this case we set nodeaffinity. We are create below pod:
```
# oc apply -f nodeaffinity.yml
# oc get pod
NAME                 READY   STATUS    RESTARTS   AGE
with-node-affinity   0/1     Pending   0          6s
```
But pod status pending:
```
oc describe pod with-node-affinity
Name:         with-node-affinity
Namespace:    myproject
Priority:     0
Node:         <none>
Labels:       <none>
Annotations:  kubernetes.io/limit-ranger:
                LimitRanger plugin set: cpu, memory request for container with-node-affinity; cpu, memory limit for container with-node-affinity
              openshift.io/scc: anyuid
Status:       Pending
IP:
IPs:          <none>
Containers:
  with-node-affinity:
    Image:      bitnami/nginx
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     20m
      memory:  20Mi
    Requests:
      cpu:        20m
      memory:     20Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-t94nw (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  default-token-t94nw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-t94nw
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  15s   default-scheduler  0/7 nodes are available: 1 node(s) had taint {node.kubernetes.io/unreachable: }, that the pod didn't tolerate, 2 node(s) didn't match Pod's node affinity, 4 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
  Warning  FailedScheduling  15s   default-scheduler  0/7 nodes are available: 1 node(s) had taint {node.kubernetes.io/unreachable: }, that the pod didn't tolerate, 2 node(s) didn't match Pod's node affinity, 4 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
```
In events we look 'nodes didnt match pods node affinity'. This mean it: Node node not set this label: 'end=dev'. Thats why we set this label:
```
# oc label node ocp-w-2.lab.ocp.lan env=dev
```
Wen i set this label, pod just begin running:
```
# oc get pod -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP             NODE                  NOMINATED NODE   READINESS GATES
with-node-affinity   1/1     Running   0          11m   10.131.1.150   ocp-w-2.lab.ocp.lan   <none>           <none>
```

# Taint and Tolerations
<!-- This look like affinity, already same logic. But in the T&T this nodes only running this which matching pods. Below we set taint in node:
```
# oc adm taint node ocp-w-2.lab.ocp.lan key1=value1:NoSchedule
node/ocp-w-2.lab.ocp.lan tainted
[root@ocp-svc managing-pod-scaling-and-scheduling]# oc describe  node ocp-w-2.lab.ocp.lan | grep -i taint
Taints:             key1=value1:NoSchedule
```
I create deployment 'mynginx':
```
# oc create deployment myginx --image=bitnami/nginx --replicas=5
deployment.apps/myginx created
[root@ocp-svc managing-pod-scaling-and-scheduling]# oc get pod -o wide
NAME                      READY   STATUS              RESTARTS   AGE   IP       NODE                  NOMINATED NODE   READINESS GATES
myginx-6bf5d88d75-9chrj   0/1     ContainerCreating   0          5s    <none>   ocp-w-1.lab.ocp.lan   <none>           <none>
myginx-6bf5d88d75-kpsv5   0/1     ContainerCreating   0          5s    <none>   ocp-w-1.lab.ocp.lan   <none>           <none>
myginx-6bf5d88d75-mjddb   0/1     ContainerCreating   0          5s    <none>   ocp-w-1.lab.ocp.lan   <none>           <none>
myginx-6bf5d88d75-tthlz   0/1     ContainerCreating   0          5s    <none>   ocp-w-1.lab.ocp.lan   <none>           <none>
myginx-6bf5d88d75-vwtrn   0/1     ContainerCreating   0          5s    <none>   ocp-w-1.lab.ocp.lan   <none>           <none>
```
Above we look all pods run 'ocp-w-1.lab.ocp.lan' node. I edit this deployment section 'tolerations': -->
