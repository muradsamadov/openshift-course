# Build s2i
Bizim githubda repomuz varki burada kod var. Bunu bir onceki dersde build etdik, oradan cixan is ile yeni deployment yaratdiq. Ammaki bunu daha rahat yolu varki, Git repoda kod var. Bu kodu s2i ile deploy etmek olur. Misalcun asagidaki repoda Dockerfile var. s2i ile ozu hem build hemde deploy edecekdir:
```
# oc new-app --name mydeployment https://github.com/muradsamadov/php-project.git --as-deployment-config
```
mydeployment adinda application yaratdiq. Eger kodda nese bir deyisiklik olaraq 'start-build' yeniden ustune deploy etmek olur:
```
# oc start-build bc/mydeployment
```
Biz bu halda githuba kod yerlesdiririk ve s2i ile application yaradiriq.
Ola biler ki, Dockerfile olmasin yalniz misalcun 'index.php' fayli olsun. Bu halda s2i avtomatik olaraq kodu basa dusur ve bu haldada application yaratmaq olur:
```
# oc new-app --name mydeployment-2 https://github.com/muradsamadov/php-project-2.git --as-deployment-config
```
Yenide 'start-build' ile repoda yeni kodda deyisiklik oldugu halda deploy etmek olar.

# Builder image 
Bele bir halda ola bilerki yuxaridaki kimi repo ola biler orada yalniz kod ola biler. s2i ozunun build-image-si var ki meselen php kodu deploy olacaqsa ozu php adinda builder-image verir:
```
# oc new-app --name mydeployment-2 php~https://github.com/muradsamadov/php-project-2.git --as-deployment-config
```
Bu halda 'php~' bu sekilde qeyd etmekle applicationu deploy etmek olur.

# s2i scripts
Bele basa dusdum ki, repodan build edende burada .s2i/bin direktoriyasinda 'assemble' adinda fayl olur ki, bu fayli container daxilinde bash script kimi calisdirir. Bu halda kod olan repoda mutleq elave olaraq .s2i/bin/assemble fayli olmalidir.
Elave olaraq qeyd edim ki, .s2i/bin/assemble faylinda yazilma ardicilligi var:
```
# cat .s2i/bin/assemble
#!/bin/sh

echo Hello from before assemble script 

# Call the original script
/usr/libexec/s2i/assemble

echo Hello from after assemble script

touch /opt/app-root/src/myfile.php
echo 'salam' > /opt/app-root/src/myfile.php
```
Yuxaridaki faylda asagidaki hisseye kimi default olaraq yazilmalidir:
```
#!/bin/sh

echo Hello from before assemble script 

# Call the original script
/usr/libexec/s2i/assemble

echo Hello from after assemble script
```
gorunduyu kimi ilk nobede bu skript ise dusmelidir '/usr/libexec/s2i/assemble', cunki bunun vasitesile ilk novbede kod deploy olur sonra ise hemin kontainerde dediyimiz kimi bunun ardi olaraq diger yazdigimiz skript calisir:
```
touch /opt/app-root/src/myfile.php
echo 'salam' > /opt/app-root/src/myfile.php
```
Bu vasitesile kod oldugu kimi deploy olur ve ayri olaraq '/opt/app-root/src/myfile.php' faylinda bu fayl yaranmis olur.
Eger bu asagidakini assemble faylinda yazmayib sadece bunu yazsaq:
```
touch /opt/app-root/src/myfile.php
echo 'salam' > /opt/app-root/src/myfile.php
```
bu halda ise kontainer daxilinde hecbir kod yaranmir yalniz burada qeyd olunan 'touch /opt/app-root/src/myfile.php' fayl yaranacaqdir, cunki burada bu skript icra olunmayib '/usr/libexec/s2i/assemble'