# oc-proxy

OC Proxy provides an interactive authentication proxy to Kubernetes clusters, using OAuth2 authentication issuer, or
non-interactive autentication, using a bearer [JWT](https://jwt.io/) (HS256).

- Proxying the Kubernetes API
- Serving frontend static assets
- Interactive user Authentication, using OAuth2 server (require JWT token using HS256 hash)
- Bearer token Authentication (require JWT using HS256 hash)
- oc-proxy can proxy WebSockets, the [noVNC](https://novnc.com/) demo shows WebSocket access to [kubevirt](https://kubevirt.io/) viertual machines noVNC server.
- oc-proxy can get an access token via [Openshifts OAuth2 server](https://docs.openshift.com/container-platform/4.7/authentication/configuring-internal-oauth.html), if this OAuth2 server is used, the proxy will not require a pre existing token to run, the server is installed by default on [OKD](https://www.okd.io/) k8s clusters.

The proxy will validate JWT bearer tokens (HS256) check for "allowedAPIMethods" and "allowedAPIRegexp" claims, and if token and request are valid,
send an API request using the proxy-known k8s token.

## Compile and run

``` bash
go build -o ./ ./cmd/oc-proxy/

./oc-proxy --help

# Run without an OAuth2 server
# This method will only support non-interactive authentication

# Run using OKD internal OAuth2 server
# This method requires OKD or Openshift cluster
oc create -f deploy/oauth-client-example.yaml
./oc-proxy \
    --api-server=<your k8s API server URL>  \
    --listen http://0.0.0.0:8080 \
    --base-address http://localhost:8080 \
    --skip-verify-tls
```

## Example

``` bash
# git clone the source and cd into the base directory.

# Build oc-proxy
go build -o ./ ./cmd/oc-proxy/

# Create an oauthclient CR for the demo
oc create -f deploy/oauth-client-example.yaml

# Creat self sighned certificate (needed if server use TLS)
openssl ecparam -genkey -name secp384r1 -out key.pem
openssl req -new -x509 -sha256 -key key.pem -out cert.pem -days 3650

# get the API server CA certificate (can be skipped by using --skip-verify-tls flag)
oc get secrets -n default --field-selector type=kubernetes.io/service-account-token -o json | jq '.items[0].data."ca.crt"' -r | python -m base64 -d > ca.crt

# For the noVNC demo clone noVNC html files, (this demo requires kubevirt installed on the server)
# we will use noVNC static html files to demo oc-proxy ability to mix static html with k8s api calls.
git clone https://github.com/novnc/noVNC web/public/noVNC

# Proxy the noVNC html files mixed with k8s API (replace the cluster with one you own)
# note that the proxy address must match the redirect address in the oauthclient CR we created
# earlier.
./oc-proxy --api-server https://api.ostest.test.metalkube.org:6443

# Browse to a VM VNC (replace the vm name and namespace to one you can access with your credentials)
# export NAMESPACE=yzamir
# export NAME=rhel7-steep-cod
# https://localhost:8080/noVNC/vnc_lite.html?path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/${NAMESPACE}/virtualmachineinstances/${NAME}/vnc
https://localhost:8080/noVNC/vnc_lite.html?path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/yzamir/virtualmachineinstances/rhel7-steep-cod/vnc
```

![alt demo gif](https://raw.githubusercontent.com/yaacov/oc-proxy/main/web/public/demo2.gif)
