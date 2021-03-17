# Steps to authenticate access to a virtual machine noVNC console (Everytime console access is required)

## 1- Set the following variables required for creating the operator CRs:
``` bash
$ vm=rhel6-150.ocp4.xxx.xxx 
$ ns=ocs-cnv
$ ocgateroute=oc-gate.apps.ocp4.xxx.xxx
$ ocgatepath=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/$ns/virtualmachineinstances/$vm/vnc
$ posturl=https://$ocgateroute/login.html
$ postpath=/noVNC/vnc_lite.html?path=$ocgatepath
$ date=$(date "+%y%m%d%H%M")
```

## 2- Inject the ocgatepath into gatetoken.yaml and create the GateToken custom resource:
$ sed -i "s|VMNAME|$vm-$date|g;s|OCGATEPOSTPATH|$ocgatepath|g" gatetoken.yaml

$ oc create -f gatetoken.yaml
``` bash
gatetoken.ocgate.yaacov.com/oc-gate-token created
```

$ bt=<bearer token>

$ apipath="https://api.ocp4.xxx.xxx:6443/apis/ocgate.yaacov.com/v1beta1/namespaces/oc-gate/gatetokens"

$ data=\'{\"apiVersion\":\"ocgate.yaacov.com/v1beta1\",\"kind\":\"GateToken\",\"metadata\":{\"name\":\"$vm-$date\",\"namespace\":\"oc-gate\"},\"spec\":{\"match-path\":\"^/$consolepath\"}}\'

$ curl -k -H 'Accept: application/json' -H \"Authorization: Bearer $bt\" -H \"Content-Type: application/json\" --request POST --data $data $apipath

## 3- Set and display the content of consoleurl:
$ token=$(oc describe gatetoken $vm-$date -n oc-gate | grep Token: | awk '{print $2}')

$ consoleurl=${posturl}?token=${token}\\&then=$postpath

$ echo $consoleurl
``` bash
https://oc-gate.apps.ocp4.xxx.xxx/login.html?token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MTU4MzA5NDQsIm1hdGNoTWV0aG9kIjoiR0VULE9QVElPTlMiLCJtYXRjaFBhdGgiOiJeL2s4cy9hcGlzL3N1YnJlc291cmNlcy5rdWJldmlydC5pby92MWFscGhhMy9uYW1lc3BhY2VzL29jcy1jbnYvdmlydHVhbG1hY2hpbmVpbnN0YW5jZXMvcmhlbDYtMTUwLm9jcDQuZ29sZG1hbi5sYWIvdm5jIiwibmJmIjoxNjE1ODI3MzQ0fQ.DXWHo5fLon-UEHpQn2D93PDR03RbFC7ANmiCwMiUeNmBhzu6mk03weDpc_irWFE5fWMUXR2dAZFpKodURiTnioCBKTHoWGX_9cneeQ-Bkqo5hhsYM4cvY4bD4EwweA_iSX6rdvyxPc50F3bgEmRLttNYBRaQyn_vTOunwxsyATnSb4ft4n9zSaSjSpaFvfVyyKFZLhf4P8ohVVve-DxpfRdVSWFK7j4xRWMLv6UqdOPTQ2g25uBpNrJM64YDQY26gWDmZGu3DprMtmxRFuCsaqrl7N1G8x_LNHx9wSc37e85zbCrnBv59Btb1wndq2bM5lT12SuFchtUwq5Hi3mNZg&then=/noVNC/vnc_lite.html?path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/ocs-cnv/virtualmachineinstances/rhel6-150.ocp4.xxx.xxx/vnc
```
![Screenshot from 2021-03-15 18-26-52](https://user-images.githubusercontent.com/77073889/111229439-47ce9980-85bc-11eb-9cb7-d0b6119c2497.png)

$ curl -k -H \'Accept: application/json\' -H \"Authorization: Bearer $bt\" $apipath/$vm-$date \| jq .status.token
