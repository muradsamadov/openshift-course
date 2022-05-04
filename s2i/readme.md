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