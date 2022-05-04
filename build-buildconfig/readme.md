# Build
Build ile git repodan Docker fayli build ede bilerik. Bu halda yeni is ve istag yaranacaqdir. Yalniz bu halda pod calismir. Sadece olaraq build edir ve bize is verir. Bu halda biz hemin isden istifade edib yeni deployment yarada bilerik. Etrafli Builds.Intro.txt ve Builds.StartBuild.txt faylinda qeyd olunub. Ola bilerki repodan nese deyisiklik olunub, bu halda start-build etmek olar.

Asagidaki komanda ile gitlab-dan Dockerfile-i build edirik. Bunu docker build-e benzetmek olar:
```
# oc new-build https://gitlab.com/practical-openshift/hello-world.git
# oc get all
NAME                      READY   STATUS    RESTARTS   AGE
pod/hello-world-1-build   1/1     Running   0          88s

NAME                                         TYPE     FROM   LATEST
buildconfig.build.openshift.io/hello-world   Docker   Git    1

NAME                                     TYPE     FROM          STATUS    STARTED              DURATION
build.build.openshift.io/hello-world-1   Docker   Git@9e4d905   Running   About a minute ago

NAME                                         IMAGE REPOSITORY                                                                TAGS   UPDATED
imagestream.image.openshift.io/golang        default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/golang        1.17   About a minute ago
imagestream.image.openshift.io/hello-world   default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/hello-world
```
Yuxaridan gorunduyu kimi application-u build etdik. Bu halda yalniz build edirik application deploy etmirik. Build edersen is avtomatik olaraq yaranir. Yuxarida qeyd olunubdur. Git repoda Dockerfile ve source kodlar var. Gedir hemin repoda olan Dockerfile build edir ve is halina salir.

Asagidaki komanda ile bc-in loglarina baxmaq olar:
```
# oc logs bc/hello-world
```

# Start Build
Ola bilerki kodda nese deyisiklik olunub ve onu yeniden deploy etmek lazimdir. Bu halda asagidaki komanda ile yeniden build edirik:
```
# oc start-build bc/hello-world
```
Build etdik ve yeni ikinci build yarandi. Bu halda is-de update olmus oldu:
```
# oc get bc
NAME          TYPE     FROM   LATEST
hello-world   Docker   Git    2
# oc get build
NAME            TYPE     FROM          STATUS     STARTED          DURATION
hello-world-1   Docker   Git@9e4d905   Complete   46 minutes ago   2m4s
hello-world-2   Docker   Git@9e4d905   Complete   18 minutes ago   1m28s
# oc get is
NAME          IMAGE REPOSITORY                                                                TAGS     UPDATED
golang        default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/golang        1.17     46 minutes ago
hello-world   default-route-openshift-image-registry.apps.lab.ocp.lan/myproject/hello-world   latest   16 minutes ago
```
bc-de  'LATEST' hissesinde 2 eded build oldugu gorsenir. 'oc get build' ederek sonuncu buildi 'hello-world-2' gormek olur.

# Start Build Git Branch
Asagidaki komanda ile git-den yalniz 'update-message' branch altinda olan kodu build edecik:
```
# oc new-build --name mybuild https://gitlab.com/practical-openshift/hello-world.git#update-message
# oc get bc/mybuild
NAME           TYPE     FROM                 LATEST
mybuild        Docker   Git@update-message   1
```

# Build Subdirectory
Asagidaki komanda ile git repodan build edir yalniz istediyin direktoriya daxilinden. Ola bilerki kod git repodadir yalniz basqa direktoriyadadir:
```
# oc new-build https://gitlab.com/practical-openshift/labs.git --context-dir hello-world
```
Yuxaridaki halda qeyd olunan repodan 'hello-world' direktoriyasindan build edir.