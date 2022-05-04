# Creating Image Stream 
public repodan yeni is yaratmaq olar ve openshift-da lokal is kimi istifade etmek olar. Etrafli ImageStream.Creating.txt faylinda gosterilib.

new-app ile yeni dc yaradanda is avtomatik ozu ozune yaradir. Bize lazimdirki conteyner deploy olunmadan is yaransin. ImageStream.Creating.txt faylinda gorunduyu kimi image-den is yaradilir:
```
# oc import-image --confirm quay.io/practicalopenshift/hello-world
imagestream.image.openshift.io/hello-world imported

Name:                   hello-world
Namespace:              myproject
Created:                Less than a second ago
Labels:                 <none>
Annotations:            openshift.io/image.dockerRepositoryCheck=2022-05-04T11:57:25Z
Image Repository:       default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/hello-world
Image Lookup:           local=false
Unique Images:          1
Tags:                   1

latest
  tagged from quay.io/practicalopenshift/hello-world

  * quay.io/practicalopenshift/hello-world@sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e
      Less than a second ago

Image Name:     hello-world:latest
Docker Image:   quay.io/practicalopenshift/hello-world@sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e
Name:           sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e
Created:        Less than a second ago
Annotations:    image.openshift.io/dockerLayersOrder=ascending
Image Size:     135.1MB in 7 layers
Layers:         2.803MB sha256:aad63a9339440e7c3e1fff2b988991b9bfb81280042fa7f39a5e327023056819
                301.3kB sha256:c732a2540651eb09faab95b03b3b5928ab23e096fae0006bdc2abf9e0cb5bfb4
                153B    sha256:7b2225181d6bcfb7877529a7257ff207cb14e52831420f770cbc2187031b6228
                132MB   sha256:c8dae7ec6990c7bc4c11880ff4c8ad988fbb8e33b575d0c4bcb89e34e68e0624
                126B    sha256:08684ee472f39cbba2db3b665d1e96eb3ba2e634302461e4bf13aada1d7f6a7e
                357B    sha256:552574c585c367b0fc9ed44570c9055e573c8bb8ffd9cb173558245d4a3fa7ec
                400B    sha256:aaed0f9cbe2b951b99827305ced09f41e7349e98415d64d3ca33aba3e1a8e720
Image Created:  2 years ago
Author:         <none>
Arch:           amd64
Command:        /bin/sh -c go run hello-world.go
Working Dir:    /go
User:           1001
Exposes Ports:  8080/tcp
Docker Labels:  io.openshift.expose-services=8080/http
Environment:    PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
                GOLANG_VERSION=1.14.1
                GOPATH=/go
                MESSAGE=Welcome! You can change this message by editing the MESSAGE environment variable.
                HOME=/go
```
Yuxarida outputuda gosterilmisdir. is yarandi:
```
# oc get is
NAME          IMAGE REPOSITORY                                                                TAGS     UPDATED
hello-world   default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/hello-world   latest   About a minute ago
```
Yuxarida gorunduyu kimi 'hello-world' adinda is yaradildi ve tag-i gosterilmisdir. Is yaradilarken openshift-de image olaraq save olunur:
```
# oc get image -A sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e
NAME                                                                      IMAGE REFERENCE
sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e   quay.io/practicalopenshift/hello-world@sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e
```
Istag-da ise image-in taglarina baxmaq olar. Hal-hazirda yalniz 'hello-world:latest' tag-i movcuddur:
```
# oc get istag
NAME                 IMAGE REFERENCE                                                                                                  UPDATED
hello-world:latest   quay.io/practicalopenshift/hello-world@sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e   4 minutes ago
```

Qeyd edim ki, artiq is yaradilmisdir. Bu is-i deploy edib dc,svc yaratmaq olar. Artiq publikden application-u deploy etmeye ehtiyac yoxdur:
```
# oc new-app myproject/hello-world:latest --as-deployment-config
# oc status
In project myproject on server https://api-int.lab.ocp.lan:6443

svc/hello-world - 172.30.29.128:8080
  dc/hello-world deploys istag/hello-world:latest
    deployment #1 deployed 20 seconds ago - 1 pod


2 infos identified, use 'oc status --suggest' to see details.
```

# Image Stream Tags
publikde olan imageden import edib yeni is yaratdiq. Hemin image-in yeni tag-da is kimi yaratmaq olur:
```
# oc tag quay.io/practicalopenshift/hello-world:update-message hello-world:update-message
# oc get is
NAME          IMAGE REPOSITORY                                                                TAGS                    UPDATED
hello-world   default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/hello-world   update-message,latest   Less than a second ago
# oc get istag
NAME                         IMAGE REFERENCE                                                                                                  UPDATED
hello-world:latest           quay.io/practicalopenshift/hello-world@sha256:2311b7a279608de9547454d1548e2de7e37e981b6f84173f2f452854d81d1b7e   2 minutes ago
hello-world:update-message   quay.io/practicalopenshift/hello-world@sha256:4e1e6736888d97170163c0ae0780625a4035ebf7fc6a2ecfa237f3f3f9f6f4c2   4 seconds ago
```
Yuxaridanda gorunduyu kimi is-de 'TAGS' in altinda 'update-message,latest' yeni yaradilan 'update-message' tag-da elave olundu. 'oc get istag' ile de baxmaq olar.

