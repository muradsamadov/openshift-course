# Managing Cluster Web Console
Asagidaki komanda ile openshift-console ns-de route-u print edirik ve qeyd olunan link ile console-a daxil olmaq olar (https://console-openshift-console.apps.lab.ocp.lan):
```
# oc get route -n openshift-console
NAME               HOST/PORT                                      PATH   SERVICES    PORT    TERMINATION          WILDCARD
console            console-openshift-console.apps.lab.ocp.lan            console     https   reencrypt/Redirect   None
downloads          downloads-openshift-console.apps.lab.ocp.lan          downloads   http    edge/Redirect        None
external-console   ocpmgr.xalqbank.az

# oc whoami --show-console
https://console-openshift-console.apps.lab.ocp.lan
```
