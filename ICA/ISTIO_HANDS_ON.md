# Hands-on preparations for ICA using `kind`

## Recap

The configuration profiles:

* `default` - istiod + ingress gateway
* `demo` - istiod, ingress and egress
* `minimal` - only control plane
* `remote` - configure remote clusters
* `empty` - deploy nothing, base profile for customizations
* `preview` - contains experimental features
* `ambient` - ambient mesh profile, istiod, CNI and ztunnel

These comes from YAML definitions, basically which components are enabled and how they are configured.

## Hands-on

1. Install the version of `istioctl` that matches the exam environment, which is `1.26.x`, let's use the latest patch version (as of writing this): `1.26.7`:

```bash
curl -L https://istio.io/downloadIstio \
    | ISTIO_VERSION=1.26.7 \
    TARGET_ARCH=x86_64 sh -
```

1. Export the `istioctl` binary to your `PATH`:

```bash
export PATH=$PATH:$(pwd)/istio-1.26.7/bin
```

1. Create a `kind` cluster:

```bash
kind create cluster --name istio
```

1. Install/uninstall Istio using the `demo` profile:

```bash
istioctl install --set profile=demo -y
istioctl uninstall --purge -y
```

1. Optionally, install Istio using `helm`:

Base chart:

```bash
helm install istio-base istio/base -n istio-system --create-namespace --set defaultRevision=default
```

Control plane:

```bash
helm install istiod istio/istiod -n istio-system
```

Istio ingress gateway:

```bash
helm install istio-ingress istio/gateway -n istio-ingress --create-namespace
```

### Customizations

Use the `customization-istio-operator.yaml` file to practice customizations using the Istio Operator.

See the following links:

* <https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/>
* <https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/#IstioComponentSetSpec>

Example:

```bash
istioctl install -f customization-istio-operator.yaml -y
```

### Traffic Management

Traffic management topics:

* Configuring Ingress and Egress Traffic
* Configuring Routing within a Service Mesh
* Defining Traffic Policies with Destination Rules
* Configuring Traffic Shifting
* Connecting In-Mesh Workloads to External Workloads and Services
* Using Resilience Features (circuit breaking, failover, outlier detection, timeouts, retries)
* Using Fault Injection

#### Ingress and Egress

Traffic flow:

```bash
External -> Ingress Gateway -> Virtual Service -> Destination Rule -> Pod(s)
```

Install the `httpbin` sample application:

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/httpbin/httpbin.yaml
```

Port-forward to the HTTP (80) port of the ingress gateway:

```bash
kubectl -n istio-system port-forward svc/istio-ingressgateway 8080:80
```

Send a request to the `httpbin` service:

```bash
curl -I http://localhost:8080/status/200 # not working, returns a 404
curl -I -H "Host: httpbin.example.com" http://localhost:8080/status/200 # works and returns a 200
```

For Egress traffic, apply the `egress-gateway-example.yaml` manifest and test access to `edition.cnn.com` through the egress gateway. The service entry is required to allow Istio to route traffic to external services.

```bash
kubectl apply -f manifests/egress-gateway-example.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/curl/curl.yaml
export SOURCE_POD=$(kubectl get pod -l app=curl -o jsonpath={.items..metadata.name})
kubectl exec "$SOURCE_POD" -c curl -- curl -sI https://www.google.com | grep  "HTTP/"; kubectl exec "$SOURCE_POD" -c curl -- curl -sI https://edition.cnn.com | grep "HTTP/"
```

**REMEMBER**: If you started the curl Pod before adding the `istio-injection=enabled` label to the namespace, you need to delete the Pod and recreate it to have the sidecar injected. Otherwise the traffic will not go through the egress gateway.

The `curl` Pod status should be (see the `2/2` READY status):

```bash
NAME                       READY   STATUS    RESTARTS   AGE
curl-56ddfb95dc-ntbzt      2/2     Running   0          2m38s
```

#### Request Routing

1. Install the `bookinfo` application:

Note that this includes:

* Four deployments: `productpage`, `details`, `reviews`, `ratings`
  * One service and service account per deployment
* One Gateway and one VirtualService for the Gateway that matches on all possible uris and lands in the productpage service
* Four DestinationRule resources, one per service, with subsets for each version

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/bookinfo/networking/destination-rule-all.yaml
```

1. Install virtual services:

Note that this includes:

* Four VirtualService resources, one per service, defining routing rules all to v1.

```bash
kubectl apply -f manifests/virtual-services.yaml
```

1. Change the review virtual service to route 100% of traffic to version v2 IF you're logged in as user `jason`, use your browser to test it. With user `john` and v2 you should see the star ratings:

Note that this changes the existing `reviews` VirtualService to add a new routing rule!

```bash
kubectl apply -f manifests/user-header-routing.yaml
```

#### Fault injection

1. Apply the fault injection configuration to the `reviews` service:

```bash
kubectl apply -f manifests/fault-injection.yaml
```

#### Traffic shifting

1. Apply the traffic shifting configuration to the `reviews` service:

```bash
kubectl apply -f manifests/traffic-shifting.yaml
```

#### Request timeouts

1. Route requests to `v2` of the `reviews` service that calls the `ratings` service:

```bash
kubectl apply -f manifests/request-timeout.yaml
```

1. Browse to the Bookinfo productpage and observe that requets to the `reviews` service time out (because `v2` of `reviews` calls `ratings` which is not deployed).

2. Apply the request timeout configuration to the `reviews` service:

```bash
kubectl apply -f manifests/request-timeout-reviews.yaml
```

### Authentication

1. Create a new `kind` cluster or cleanup the existing one! If you create a new one use the `default` profile.

```bashkind delete cluster --name istio
kind create cluster --name istio-authnz
istioctl install --set profile=default -y
```

1. Create and enable automatic sidecar injection for the namespaces needed:

```bash
kubectl create ns foo
kubectl label namespace foo istio-injection=enabled
kubectl create ns bar
kubectl label namespace bar istio-injection=enabled
kubectl create ns legacy
```

1. Install the `httpbin` and `curl` sample applications in all three namespaces:

```bash
kubectl apply -n foo -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/httpbin/httpbin.yaml
kubectl apply -n foo -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/curl/curl.yaml
kubectl apply -n bar -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/httpbin/httpbin.yaml
kubectl apply -n bar -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/curl/curl.yaml
kubectl apply -n legacy -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/httpbin/httpbin.yaml
kubectl apply -n legacy -f https://raw.githubusercontent.com/istio/istio/refs/heads/master/samples/curl/curl.yaml
```

1. Test reachability:

```bash
kubectl exec "$(kubectl get pod -l app=curl -n bar -o jsonpath={.items..metadata.name})" -c curl -n bar -- curl http://httpbin.foo:8000/ip -s -o /dev/null -w "%{http_code}\n"
```

1. Test all reachability combinations:

```bash
for from in "foo" "bar" "legacy"
do 
    for to in "foo" "bar" "legacy"
    do 
        kubectl exec "$(kubectl get pod -l app=curl -n ${from} -o jsonpath={.items..metadata.name})" -c curl -n ${from} -- curl -s "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "curl.${from} to httpbin.${to}: %{http_code}\n"; 
    done 
done
```

#### Auto mutual TLS

By default Istio tracks the server workloads migrated to Istio proxies and configures client proxies to send mutual TLS traffic to those workloads automatically, and **plain text** traffic to workloads **without** Istio sidecars.

It uses mTLS without you doint anything! When using mTLS the proxy injects the `X-Forwarded-Client-Cert` header to the upstream request to the backend.

Check with:

```bash
kubectl exec "$(kubectl get pod -l app=curl -n foo -o jsonpath={.items..metadata.name})" -c curl -n foo -- curl -s http://httpbin.foo:8000/headers -s | jq '.headers["X-Forwarded-Client-Cert"][0]' | sed 's/Hash=[a-z0-9]*;/Hash=<redacted>;/'
```

Also check that the legacy namespace is not using mTLS, since we're not injecting sidecars there:

```bash
kubectl exec "$(kubectl get pod -l app=curl -n foo -o jsonpath={.items..metadata.name})" -c curl -n foo -- curl http://httpbin.legacy:8000/headers -s | grep X-Forwarded-Client-Cert
```

Now globally enable mTLS STRICT mode for all namespaces:

```bash
kubectl apply -f manifests/peer-auth-strict.yaml
```

_Assumes `istio-system` is the _root namespace_.

Now when you run the test with reachability combinations again, the calls to the legacy namespace should fail with connection errors, since those workloads are not using Istio sidecars and mTLS is enforced.

Clean up before moving on:

```bash
kubectl delete peerauthentication -n istio-system default
```

#### Enable mTLS per namespace or workload

Now enable mTLS using a Namespace-wide policy:

```bash
kubectl apply -f manifests/ns-peer-auth-strict.yaml
```

Now when you now run the tests with reachability combinations again, the calls to the `foo` namespace should still fail with connection errors, since those workloads are not using Istio sidecars and mTLS is enforced in the `foo` namespace.

#### Enable mTLS per workload

1. Enable strict mTLS for the `httpbin` service in the `bar` namespace only:

```bash
kubectl apply -f manifests/workload-peer-auth-strict.yaml
```

Requests from `curl.legacy` to `httpbin.bar` starts failing.

1. Use the `portLevelMtls` setting in the PeerAuthentication to enable mTLS only on specific ports. Apply the `workload-peer-auth-strict-port-level.yaml` manifest to enable mTLS only on port `8080` for the `httpbin` service in the `bar` namespace:

```bash
kubectl apply -f manifests/disable-mtls-port-level.yaml
```

Now when you run the tests that `curl.legacy` can reach `httpbin.bar` on port `8000` but not on port `8080` which we have enabled mTLS for.

REMEMBER: Workload-specific PeerAuthentication policy takes precedence over a namespace-wide policy!

#### Authorization

Switch back to the other `kind` cluster you used before for traffic management.

Configure accesss control for workloads using HTTP traffic:

1. Apply the `allow-nothing` policy in the `default` namespace:

```bash
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: default
spec:
  {}
EOF
```

Browse to the Bookinfo productpage and observe:

```
RBAC: access denied
```

1. Create a `productpage-viewer` policy to allow access with `GET` method to the `productpage` workload:

```bash
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "productpage-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  action: ALLOW
  rules:
  - to:
    - operation:
        methods: ["GET"]
EOF
```

Now you should be able to browse to the Bookinfo productpage again but the details and reviews pages returns errors. These are expected since we have not granted the `productpage` workload access to the `details` and `reviews` workloads yet.

1. Create the `details-viewer` policy to allow the `productpage` workload using a specific svc account to access `details` workload through `GET` methods:

```bash
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "details-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: details
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
EOF
```

1. Create a similar one for the `reviews` workload:

```bash
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "reviews-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
EOF
```
