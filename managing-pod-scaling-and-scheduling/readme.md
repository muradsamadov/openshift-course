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