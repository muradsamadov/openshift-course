# Adding roles to users
Kubernetesde oldugu kimi openshitde-de role ve clusterrole anlayisi eynidir. Bu misalda var olan 'admin' role-na 'anna' userini elave edecem. Bu icaze yalniz 'rbac-test-project' projekti ucun olacaqdir. Daha sonra hemin rolu describe edib emin olacamki komanda set olunubdur. Ilk once 'anna' adinda user yaradiriq ve secreti deploy edirik. Etrafli melumat 'configuring-authentication' direktoriyasinda gosterilmisdir.
Asagida gorunduyu kimi qurulan openshift clusterinde user yaradilmasi ucun biz secret-den istifade edirik. Default olaraq menim clusterimde yeni yaradilan credential 'secret/htpasswd-users' olaraq bu projecte 'openshift-config' deploy edilecekdir. Ilk novbede 'extract' ile hemin secreti locala fayl kimi elave edirik. 'htpasswd' ile 'anna' adinda user yaradib hemin fayla elave edirik.Gorunduyu kimi 'cat /root/htpasswd' edib emin olmaq olarki anna useri elave edildi.Daha sonra hemin fayli secrete yeniden set edib deploy edirik. 'oc get pod -n openshift-authentication' komandasi ile podlarin yeniden deploy oldugunu goruruk. Bu halda 'anna' useri ile login olub var olan userleri list edirik. Belelikle 'anna' useri yaranmis oldu.
```
# oc extract secret/htpasswd-users -n openshift-config  --to /root --confirm
# htpasswd -b /root/htpasswd anna password
Adding password for user anna
# cat /root/htpasswd
admin:$apr1$EPD4DFFo$u38st1HrJbSmcGhusezL91
manager:$apr1$usBDPWhR$rsqNJ.Dy2eIabyLzf5cfl1
developer:$apr1$jisfm8A8$jYZs62P8hzcd6RvQKvEsE1
joni:$apr1$LSwoCwVv$LgWlNC1nDXxDTAX3f5n0G0
alex:$apr1$1AQXU12E$ZBZn3ajfaGINH1ZheycTm0
brhasanli:$apr1$NESymfBP$h8gggJPjJDAxCB/JATeRK1
anna:$apr1$KCNaBcIS$t65NSC0kBABRUrqLwE98P.
# oc set data secret/htpasswd-users -n openshift-config --from-file htpasswd=/root/htpasswd
#  oc get pod -n openshift-authentication
NAME                               READY   STATUS        RESTARTS   AGE
oauth-openshift-5d8d4968b4-jcfxk   1/1     Terminating   0          47m
oauth-openshift-5d8d4968b4-vddlg   1/1     Terminating   0          47m
oauth-openshift-d75459497-h66tf    1/1     Running       0          22s
oauth-openshift-d75459497-tmzbz    1/1     Running       0          16s
# oc login -u anna -p password
Login successful.

You have access to 102 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
[root@ocp-svc ~]# oc get user
NAME        UID                                    FULL NAME   IDENTITIES
anna        be758249-aa7d-4693-8ace-8995ea4f0a57               htpasswd-access:anna
```

'Anna' userini yaradandan sonra ona icaze vermek lazimdir. Bu halda men yeni 'rbac-test-project' adinda project yaradiram. 'anna' userinde bu projecte 'admin' role-u verirem:
```
# oc new-project rbac-test-project
# oc adm policy add-role-to-user admin anna -n rbac-test-project
# oc describe rolebinding admin -n  rbac-test-project
Name:         admin
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  admin
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  anna
[root@ocp-svc 
# oc login -u anna -p password
```
Task gele bilerki 'linda' userini view ve ya edit icazesi vermek lazimdir. 'view' yalniz list menasindadir, 'edit' ise pod yarada me modify menasindadir. Misal ucun:
```
# oc policy add-role-to-user view linda
# oc get rolebindings -o wide
NAME                    ROLE                               AGE   USERS     GROUPS                             SERVICEACCOUNTS
admin                   ClusterRole/admin                  25h   manager
system:deployers        ClusterRole/system:deployer        25h                                                myproject/deployer
system:image-builders   ClusterRole/system:image-builder   25h                                                myproject/builder
system:image-pullers    ClusterRole/system:image-puller    25h             system:serviceaccounts:myproject
view                    ClusterRole/view                   72s   linda
```
Yuxaridaki misala esasen 'linda' userinde project daxilinde 'view' icazesi veririk.

# Managing Security Context Constraints
A Security Context Constraints(SCC) similar to the Kubernetes Security Context resource, that restricts access to resources. When you are deploy docker container root privileges, this cntainer dont run because this container run root provileges. In openshift pod run only non-root privileges. Thats why we are set SCC to resolve this. In an Openshift container run selinux and with SCC we are set permission this container. SCC are available to control:
1. Running privileges container
2. Changing the user ID
3. Changing Selinux context of container
4. Using host directories as volumes.
First we are deploy new deployment s2i to github. In github are Dockerfile and s2i deploy this file.Process below:
```
# oc new-app --name mydeployment https://github.com/muradsamadov/php-project.git --as-deployment-config
# oc get pod
NAME                    READY   STATUS             RESTARTS   AGE
mydeployment-1-build    0/1     Completed          0          37s
mydeployment-1-deploy   1/1     Running            0          11s
mydeployment-1-f862x    0/1     CrashLoopBackOff   1          8s
[root@ocp-svc ~]# oc logs mydeployment-1-f862x
2022/05/19 07:28:41 [warn] 8#8: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2022/05/19 07:28:41 [emerg] 8#8: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
[root@ocp-svc ~]#
```
Above we are look 'mydeployment-1-f862x' pod not running because not permission. This container run root user thats why we set SCC:
```
# oc get dc  mydeployment -o yaml | oc adm policy scc-subject-review -f -
RESOURCE                        ALLOWED BY
DeploymentConfig/mydeployment   anyuid
[root@ocp-svc ~]# oc create sa nginx-sa
serviceaccount/nginx-sa created
[root@ocp-svc ~]# oc adm policy add-scc-to-user anyuid -z nginx-sa
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "nginx-sa"
[root@ocp-svc ~]# oc get sa nginx-sa
NAME       SECRETS   AGE
nginx-sa   2         34s
[root@ocp-svc ~]# oc set sa  dc mydeployment nginx-sa
deploymentconfig.apps.openshift.io/mydeployment serviceaccount updated
[root@ocp-svc ~]# oc get pod
NAME                    READY   STATUS        RESTARTS   AGE
mydeployment-1-build    0/1     Completed     0          4m9s
mydeployment-1-f862x    0/1     Terminating   5          3m40s
mydeployment-2-deploy   0/1     Completed     0          11s
mydeployment-2-lb6q2    1/1     Running       0          7s
[root@ocp-svc ~]# oc logs mydeployment-2-lb6q2
2022/05/19 07:32:12 [notice] 8#8: using the "epoll" event method
2022/05/19 07:32:12 [notice] 8#8: nginx/1.21.6
2022/05/19 07:32:12 [notice] 8#8: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/05/19 07:32:12 [notice] 8#8: OS: Linux 5.13.12-200.fc34.x86_64
2022/05/19 07:32:12 [notice] 8#8: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/05/19 07:32:12 [notice] 8#8: start worker processes
2022/05/19 07:32:12 [notice] 8#8: start worker process 9
2022/05/19 07:32:12 [notice] 8#8: start worker process 10
2022/05/19 07:32:12 [notice] 8#8: start worker process 11
2022/05/19 07:32:12 [notice] 8#8: start worker process 12
```
Now we are this pod access to selinux and pod running true privileges.