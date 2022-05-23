# Creating Self-signed Certificates
This task i want to connect servers in browser with certificate. First i create certificate and authorize it as below:
```
# openssl genrsa -des3 -out myCA.key 2048
# openssl req -x509 -new -nodes -key myCA.key -sha256 -days 3650 -out myCA.pem
# openssl genrsa  -out tls.key 2048
# openssl req -new -key tls.key -out tls.csr
# openssl x509 -req -in tls.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out tls.crt -days  1650 -sha256
```
Above the command when i set second command i set 'CommonName' like this 'mydeployment-myproject.apps.lab.ocp.lan'. Because i work project name 'myproject', i create deployment name 'mydeployment'. This name must be similar to roue.
Certificates uses 3 ways:
1. edge route: this variant .crt and .key files save on edge route:
cliet --> edge route(.crt , .key files) --> app
2. pass-through: this variant certificates save app:
client --> pass-through --> app(.crt , .key files)
3. re-encrypt:
client --> re-encrypt(cert) --> app(cert)
Below command i create 'new-app' and i create edge route:
```
# oc new-app --docker-image=bitnami/nginx --name mydeploymentoc new-app --docker-image=bitnami/nginx --name mydeployment
# oc create route edge mydeployment --service mydeployment --cert=tls.crt --key=tls.key --ca-cert=myCA.pem
# oc get route
NAME           HOST/PORT                                 PATH   SERVICES       PORT       TERMINATION   WILDCARD
mydeployment   mydeployment-myproject.apps.lab.ocp.lan          mydeployment   8080-tcp   edge          None
```
Finally i test it:
```
# curl -s -k https://mydeployment-myproject.apps.lab.ocp.lan
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
