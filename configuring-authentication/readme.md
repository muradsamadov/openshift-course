# Creating and Deleting Users
Asagida 'htpasswd' komandasi ile /tmp/htpasswd faylinda username ve parol yaradiriq:
```
# htpasswd -c -B -b /tmp/htpasswd admin password
# htpasswd  -b /tmp/htpasswd anna password
# htpasswd  -b /tmp/htpasswd linda password
# cat /tmp/htpasswd 
admin:$2y$05$0rYdUTAC2IQPp4KL5IKMrudMJGtP6.xICoVH8jfb4fIC4m9N4QEPq
anna:$apr1$OqEcrmba$q5962d0fuS/MtTJGrACrX/
linda:$apr1$0Mi2grJi$k/7IKviqrDJEz97vRuxEc/
```
Yuxarida '/tmp/htpasswd' faylinda gorunduyu kimi username ve parol sifreli sekilde yaradildi.
Asagidaki komanda ile openshift-config ns-de htpasswd-secret adinda secret yaradiriq:
```
# oc create secret generic htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config
```
Asagidaki komanda ile 'anna' useridi cluster-admin edirik
```
# oc adm policy add-cluster-role-to-user cluster-admin anna
```
Daha sonra yaradilan 'htpasswd-secret' secreti oauth-a set edirik, name-i deyisdirib 'htpasswd-secret':
```
# oc edit oauth
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: htpasswd-secret
```
Yuxaridaki komandanin deploy olun-olmadigini bu ns-de pod-u list etmekle emin olmaq olar:
```
# oc get pod -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-7b7fc6b6b4-4c47q   1/1     Running   0          84m
oauth-openshift-7b7fc6b6b4-ntjx7   1/1     Running   0          84m
```
Asagidaki komanda ile openshift-e anna useri ile login oluruq:
```
# oc login -u anna -p password
```
Anna cluster-admin permission-dadir. linda useri ise sade userdir ve hecneyi list ede bilmir:
```
# oc login -u linda -p password
# oc get users
NAME        UID                                    FULL NAME   IDENTITIES
admin       0ab7ef20-49cb-4a0f-8e72-730b59cf4257               htpasswd-access:admin
anna        86a4f74c-5699-4250-9563-eeaaacb72ef1               htpasswd-access:anna
developer   9a7a29f8-f8b2-4e44-bedd-39686e4cf404               htpasswd-access:developer
linda       296905ce-50ad-4d27-94f2-5a463df5c1be               htpasswd-access:linda
```
'oc get users' komandasi ile movcud olan userleri list edirik.

# Managing Users
Ola bilerki yeni user yaradilsin ve ya userin parolu deyisilsin. Bu halda secret-de deyisiklik olunmalidir. Bu halda asagidaki kimi secreti extract edirik /root/htpasswd faylina, sonra ise onu set edirik:
```
# oc extract secret/htpasswd-secret --to /root --confirm
# cat /root/htpasswd
admin:$2y$05$0rYdUTAC2IQPp4KL5IKMrudMJGtP6.xICoVH8jfb4fIC4m9N4QEPq
anna:$2y$05$pGghSls2VisUWtGfvvblpOxPhuhxdljHuBTjNlR5PkJha7/QXM0FW
linda:$apr1$0Mi2grJi$k/7IKviqrDJEz97vRuxEc/
htpasswd -b -B /root/htpasswd anna password
htpasswd -b -B /root/htpasswd bob password
# oc set data secret/htpasswd-secret --from-file htpasswd=/root/htpasswd
```
Yuxarida gorunduyu kimi anna-nin parolunu deyisdik ve bob adinda yeni user yaratdiq:
```
# cat /root/htpasswd
admin:$2y$05$0rYdUTAC2IQPp4KL5IKMrudMJGtP6.xICoVH8jfb4fIC4m9N4QEPq
anna:$2y$05$pGghSls2VisUWtGfvvblpOxPhuhxdljHuBTjNlR5PkJha7/QXM0FW
linda:$apr1$0Mi2grJi$k/7IKviqrDJEz97vRuxEc/
bob:$2y$05$qt5ncmp5TVJo3aNcpCCPqeIbLONUkp3BwhFRWASRw9fGD3KUN6pSC
```

# Managing User Group Membership
developers adinda qrup yaradaq ve 'linda' userini hemin qrupa elave edek:
```
# oc adm groups new developers
# oc adm groups add-users developers linda
# oc get groups
NAME         USERS
developers   linda
oc policy add-role-to-group edit developers
```
'oc policy add-role-to-group edit developers' komandasi ile policye grant privilege verirem