# ambient-mesh

**Sidecar-less data plane for istio

![image](https://user-images.githubusercontent.com/119640027/205217906-f885cb44-a272-4426-8c62-184bdb7bb325.png)


## Ambient Mesh Demo


**Download istioctl for ambient:**

- __[wget](https://storage.googleapis.com/istio-build/dev/0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82/istioctl-0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82-linux-arm64.tar.gz)__ - LINUX ARM64

- __[wget](https://storage.googleapis.com/istio-build/dev/0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82/istioctl-0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82-linux-amd64.tar.gz)__ - LINUX AMD64

- __[wget](https://storage.googleapis.com/istio-build/dev/0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82/istioctl-0.0.0-ambient.191fe680b52c1754ee72a06b3e0d3f9d116f2e82-osx-arm64.tar.gz)__ - OSX ARM64

```
Uncompress the archive:

 #tar zxvf istioctl*.tar.gz

**The Ambient profile is designed to help you get started with Ambient easily.**

### Install Istio with the ambient profile on your Kubernetes cluster.


#./istioctl install -y --set profile=ambient

After running the above command, you’ll get the following output that indicates these four components are installed successfully!

// Some comments
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ CNI installed
✔ Installation complete
Making this installation the default for injection and validation.
By default, the Ambient profile has the Istio core, Istiod, ingress gateway, ztunnel and CNI plugin enabled. The Istio CNI plugin is responsible for detecting which application pods are part of ambient and configuring the traffic redirection between them and the ztunnel.
```

```
Run the following command to check the Pods deployed in the istio-system namespace:

chauhai1@QCV925JTJ7 ambient-mesh % kubectl get po -n istio-system -o wide ;kubectl get po -n kube-system -o wide |grep -i cni

NAME                                    READY   STATUS    RESTARTS   AGE
istio-ingressgateway-57c74cf745-lv75x   1/1     Running   0          35h
istiod-7f57456bbd-qzjh8                 1/1     Running   0          35h
ztunnel-d7rjp                           1/1     Running   0          29h
ztunnel-fnwkn                           1/1     Running   0          29h
ztunnel-mdqzk                           1/1     Running   0          35h
ztunnel-qc4c4                           1/1     Running   0          35h
ztunnel-qz6qx                           1/1     Running   0          29h
ztunnel-tkhtm                           1/1     Running   0          35h

chauhai1@QCV925JTJ7 ambient-mesh % kubectl get po -n kube-system |grep -i cni
istio-cni-node-54m8f                                     1/1     Running   0          29h
istio-cni-node-879d8                                     1/1     Running   0          35h
istio-cni-node-gftxw                                     1/1     Running   0          29h
istio-cni-node-hn6dj                                     1/1     Running   0          35h
istio-cni-node-x2fgj                                     1/1     Running   0          35h
istio-cni-node-zvpvb                                     1/1     Running   0          29h

You’ll notice the following pods are deployed in the istio-system namespace with the default Ambient profile.
```

### Deploy Sample Services (Sample Application) ###


**We're going to deploy 3 services:**

+ web-api
+ recommendation
+ purchase-history 

1. The web-api service calls the recommendation service via HTTP.
2. recommendation service calls the purchase-history service, also via HTTP.
3. We will use curl as your client, either outside of the cluster or in the cluster from the sleep pod. 
4. Pod anti-affinity rules are specified in the sleep, web-api, recommendation and purchase-history deployment descriptors to avoid them deployed on the same node whenever possible.

![image](https://user-images.githubusercontent.com/119640027/205218217-2234c519-9c6d-4818-8e70-5d70be017835.png)

**Application without Istio**

*Deploy the sample application:*

### Step -1 

 Create a namespace with name "anz-ambient-demo"

```
% kubectl create ns anz-ambient-demo
namespace/anz-ambient-demo created

% kubectl describe ns anz-ambient-demo
Name:         anz-ambient-demo
Labels:       kubernetes.io/metadata.name=anz-ambient-demo
Annotations:  <none>
Status:       Active
```
### Step -2

 Deploy the sample application in namespace "anz-ambient-demo"

```
kubectl apply -f data/steps/deploy-sample-services
serviceaccount/notsleep created
service/notsleep created
deployment.apps/notsleep created
serviceaccount/purchase-history created
deployment.apps/purchase-history-v1 created
service/purchase-history created
serviceaccount/recommendation created
deployment.apps/recommendation created
service/recommendation created
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
serviceaccount/web-api created
service/web-api created
deployment.apps/web-api created
```
### Step -3

 Run the following command to check the Pods deployed in the anz-ambient-demo namespace:

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl get po -n anz-ambient-demo
NAME                                   READY   STATUS    RESTARTS   AGE
notsleep-6db55f5cfd-k9pvx              1/1     Running   0          13s
purchase-history-v1-54c8956877-tdv6p   1/1     Running   0          13s
recommendation-7f66565d54-h9n27        1/1     Running   0          12s
sleep-67f77477d5-8c7g2                 1/1     Running   0          12s
web-api-7757bdc884-2f7xv               1/1     Running   0          11s
chauhai1@QCV925JTJ7 ambient-mesh % 
```
### Step -4

 Check the deployed application, if working as expected or not.

```
kubectl exec deploy/sleep -- curl http://web-api:8080/

You should get something like this:

chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec deploy/sleep -- curl http://web-api:8080/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1104  100  1104    0     0  55200      0 --:--:-- --:--:-- --:--:-- 55200
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.116.5.10"
  ],
  "start_time": "2022-11-15T07:45:20.808573",
  "end_time": "2022-11-15T07:45:20.822901",
  "duration": "14.328322ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.116.4.9"
      ],
      "start_time": "2022-11-15T07:45:20.814277",
      "end_time": "2022-11-15T07:45:20.821850",
      "duration": "7.572325ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.116.3.8"
          ],
          "start_time": "2022-11-15T07:45:20.819974",
          "end_time": "2022-11-15T07:45:20.820655",
          "duration": "680.444µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
``` 
### Step -5

  Add Services to Ambient Mesh via adding appropriate annotation on top of namespace <anz-ambient-demo>

*Adding services to Ambient is very simple.*

**You just need to add the istio.io/dataplane-mode=ambient label to your namespace to have all the corresponding pods managed by Ambient.**

*kubectl label namespace anz-ambient-demo istio.io/dataplane-mode=ambient*


chauhai1@QCV925JTJ7 ambient-mesh% kubectl label namespace anz-ambient-demo istio.io/dataplane-mode=ambient
namespace/default labeled

**Now, take a look at the logs of the Istio CNI DaemonSet using the following command:**

*kubectl -n kube-system logs -l k8s-app=istio-cni-node*

You should see something similar to this:

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl -n kube-system logs -l k8s-app=istio-cni-node --tail 1          
2022-11-15T07:42:26.494021Z     info    install CNI config file /host/etc/cni/net.d/10-containerd-net.conflist exists. Proceeding.
2022-11-15T07:42:26.494450Z     info    install Created CNI config /host/etc/cni/net.d/10-containerd-net.conflist
2022-11-15T07:42:26.494466Z     info    install CNI configuration and binaries reinstalled.
2022-11-15T07:56:24.992614Z     info    ambient Reconciling namespace anz-ambient-demo
2022-11-15T07:56:24.993002Z     info    ambient Namespace anz-ambient-demo is enabled in ambient mesh
2022-11-15T07:42:05.862415Z     info    cni     istio-cni cmdAdd podName: web-api-7757bdc884-2f7xv podIPs: [{IP:10.116.5.10 Mask:ffffff00}]
2022-11-15T07:42:05.862499Z     info    cni     Pod anz-ambient-demo/web-api-7757bdc884-2f7xv excluded because it only has 1 containers
2022-11-15T07:56:24.992816Z     info    ambient Reconciling namespace anz-ambient-demo
2022-11-15T07:56:24.993003Z     info    ambient Namespace anz-ambient-demo is enabled in ambient mesh
2022-11-15T07:56:24.993369Z     info    ambient Adding pod 'web-api-7757bdc884-2f7xv/anz-ambient-demo' (cdd4bbe3-7b9a-4b17-ae98-0b1172517f42) to ipset
2022-11-15T07:56:24.999836Z     info    ambient Adding route for web-api-7757bdc884-2f7xv/anz-ambient-demo: [table 100 10.116.5.10/32 via 192.168.126.2 dev istioin src 10.116.5.1]
2022-11-15T07:56:25.003370Z     info    ambient Adding pod 'notsleep-6db55f5cfd-k9pvx/anz-ambient-demo' (89c46f6b-459f-4701-a8fa-f0593efe583b) to ipset
2022-11-15T07:56:25.009739Z     info    ambient Adding route for notsleep-6db55f5cfd-k9pvx/anz-ambient-demo: [table 100 10.116.5.8/32 via 192.168.126.2 dev istioin src 10.116.5.1]
2022-11-15T07:56:25.013488Z     info    ambient Adding pod 'sleep-67f77477d5-8c7g2/anz-ambient-demo' (99ecf6d6-b14b-4ec8-94fa-8ca1b97e7ceb) to ipset
2022-11-15T07:56:25.022027Z     info    ambient Adding route for sleep-67f77477d5-8c7g2/anz-ambient-demo: [table 100 10.116.5.9/32 via 192.168.126.2 dev istioin src 10.116.5.1]
      "kubeconfig": "/etc/cni/net.d/ZZZ-istio-cni-kubeconfig",
      "cni_bin_dir": "/home/kubernetes/bin",
      "exclude_namespaces": [ "istio-system", "kube-system" ]
  }
}
2022-11-15T07:37:23.099240Z     info    install CNI config file /host/etc/cni/net.d/10-containerd-net.conflist exists. Proceeding.
2022-11-15T07:37:23.100228Z     info    install Created CNI config /host/etc/cni/net.d/10-containerd-net.conflist
2022-11-15T07:37:23.100245Z     info    install CNI configuration and binaries reinstalled.
2022-11-15T07:56:24.992639Z     info    ambient Reconciling namespace anz-ambient-demo
2022-11-15T07:56:24.992771Z     info    ambient Namespace anz-ambient-demo is enabled in ambient mesh
      "kubeconfig": "/etc/cni/net.d/ZZZ-istio-cni-kubeconfig",
      "cni_bin_dir": "/home/kubernetes/bin",
      "exclude_namespaces": [ "istio-system", "kube-system" ]
  }
}
2022-11-15T07:42:24.966313Z     info    install CNI config file /host/etc/cni/net.d/10-containerd-net.conflist exists. Proceeding.
2022-11-15T07:42:24.967250Z     info    install Created CNI config /host/etc/cni/net.d/10-containerd-net.conflist
2022-11-15T07:42:24.967337Z     info    install CNI configuration and binaries reinstalled.
2022-11-15T07:56:24.993126Z     info    ambient Reconciling namespace anz-ambient-demo
2022-11-15T07:56:24.993306Z     info    ambient Namespace anz-ambient-demo is enabled in ambient mesh
2022-11-15T07:42:04.454890Z     info    cni     istio-cni cmdAdd args: IgnoreUnknown=1;K8S_POD_NAMESPACE=anz-ambient-demo;K8S_POD_NAME=purchase-history-v1-54c8956877-tdv6p;K8S_POD_INFRA_CONTAINER_ID=a91aaa91cfc30baee14b30a8b69147b44e5c0e5d0d7f4fa714165adde7d36946;K8S_POD_UID=05768430-95aa-4a60-8988-be70fe737e74
2022-11-15T07:42:04.454894Z     info    cni     istio-cni cmdAdd with k8s args: {CommonArgs:{IgnoreUnknown:true} IP:<nil> K8S_POD_NAME:purchase-history-v1-54c8956877-tdv6p K8S_POD_NAMESPACE:anz-ambient-demo K8S_POD_INFRA_CONTAINER_ID:a91aaa91cfc30baee14b30a8b69147b44e5c0e5d0d7f4fa714165adde7d36946}
2022-11-15T07:42:04.454899Z     info    cni     ambientConf.Mode: DEFAULT
2022-11-15T07:42:04.454927Z     info    cni     ambientConf.ZTunnelReady: true
2022-11-15T07:42:04.454932Z     info    cni     istio-cni cmdAdd podName: purchase-history-v1-54c8956877-tdv6p podIPs: [{IP:10.116.3.8 Mask:ffffff00}]
2022-11-15T07:42:04.454938Z     info    cni     Pod anz-ambient-demo/purchase-history-v1-54c8956877-tdv6p excluded because it only has 1 containers
2022-11-15T07:56:24.992971Z     info    ambient Reconciling namespace anz-ambient-demo
2022-11-15T07:56:24.993151Z     info    ambient Namespace anz-ambient-demo is enabled in ambient mesh
2022-11-15T07:56:25.025426Z     info    ambient Adding pod 'purchase-history-v1-54c8956877-tdv6p/anz-ambient-demo' (05768430-95aa-4a60-8988-be70fe737e74) to ipset
2022-11-15T07:56:25.271800Z     info    ambient Adding route for purchase-history-v1-54c8956877-tdv6p/anz-ambient-demo: [table 100 10.116.3.8/32 via 192.168.126.2 dev istioin src 10.116.3.1]
```

**The CNI DaemonSet on each node constantly watches for pods (on the same node) that are running in namespaces with the istio.io/dataplane-mode=ambient label.**

| Description |
| ------------ |
- Whenever a pod with the matching namespace label is detected, the CNI DaemonSet checks if the pod opts out of ambient. If so, the CNI DaemonSet ignores the pod. If not, the CNI DaemonSet applies iptables redirect rules to redirect all the outgoing and incoming traffic from/to the pod to the ztunnel running on the same node.

# Demo - ztunnel in action (Layer 4 features of istio) #

**List of features for demo:**

1. *mTLS using ztunnel*
2. *L4 Authorization policies*

## To demo this - 

![image](https://user-images.githubusercontent.com/119640027/205218519-9064db3b-8845-4f56-bdde-baaffea4c254.png)
 
**What we do to test the ztunnel in action?**

+ The sleep (client) pod calls the web-api (server) service via http on port 8080.
+ When the sleep pod makes the request, it will be redirected to ztunnel running on the same node as the sleep pod.
+ This ztunnel will then establish an HTTP/2 CONNECT tunnel with the ztunnel running on the same node as the web-api pod and send the request as byte streams through the tunnel.
+ The target ztunnel will finally forward the request to the web-api pod running on the same node.


**What will happen in backend with ztunnel?**
```
1. Similar to sidecar, each service account will be assigned its own identity and client key/certificate pair.

2. Client ztunnel will establish the mTLS connection using the client’s key/certificate pair on behalf of the client pod and server ztunnel will terminate the mTLS connection and forward the request to the server pod.

3. By adding your client and server pods to ambient, it enables your client pod to communicate with the server pod with encrypted traffic using cryptographic identity automatically, without any code change or sidecar injection to the client or server.

4. Further, you automatically gather your pods’ key Layer 4 (L4) metrics, such as TCP bytes sent.
```
**Let's generate some traffic:**

```
kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s -I http://web-api:8080/; done'
```

You can confirm ztunnels are mediating the traffic between client and server by reviewing the ztunnel access logs.
```
kubectl -n istio-system logs -l app=ztunnel | grep '^\['
```
```
You should get something like this:
chauhai1@QCV925JTJ7 ambient-mesh % kubectl -n istio-system logs -l app=ztunnel | grep '^\['
[2022-11-15T09:15:41.834Z] "- - -" 0 - - - "-" 77 133 10 - "-" "-" "-" "-" "10.116.5.10:15088" outbound_tunnel_clus_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep 10.116.5.2:46736 10.116.5.10:8081 envoy://internal_client_address/ - - outbound tunnel
[2022-11-15T09:15:41.834Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41986 - - outbound capture listener
[2022-11-15T09:15:41.834Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41986 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.853Z] "- - -" 0 - - - "-" 125 826 6 - "-" "-" "-" "-" "10.116.4.9:15008" outbound_tunnel_clus_spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api 10.116.5.2:60216 10.116.4.9:8080 envoy://internal_client_address/ - - outbound tunnel
[2022-11-15T09:15:41.853Z] "- - -" 0 - - - "-" 125 826 7 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api/10.116.4.9:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api_to_http_recommendation.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.201:8080 10.116.5.10:56824 - - outbound capture listener
[2022-11-15T09:15:41.853Z] "- - -" 0 - - - "-" 125 826 7 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api/10.116.4.9:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api_to_http_recommendation.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.201:8080 10.116.5.10:56824 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.850Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 77 133 11 - "-" "-" "a24a3116-2b2c-4753-be33-0d2678e64c7d" "10.116.5.10:8081" "10.116.5.10:8081" virtual_inbound 10.116.5.2:39401 10.116.5.10:15088 10.116.5.2:46728 - - inbound hcm
[2022-11-15T09:15:41.850Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "10.116.5.10:15088" outbound_tunnel_clus_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep 10.116.5.2:46728 10.116.5.10:8081 envoy://internal_client_address/ - - outbound tunnel
[2022-11-15T09:15:41.850Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41992 - - outbound capture listener
[2022-11-15T09:15:41.850Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41992 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.701Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "b3f5b20e-61ba-4523-ba41-de04d4de0ff2" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:55557 10.116.3.8:15008 10.116.4.4:33730 - - inbound hcm
[2022-11-15T09:15:41.718Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "5bc75418-a4ca-48ea-8981-4ff68613d8ea" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:57475 10.116.3.8:15008 10.116.4.4:33732 - - inbound hcm
[2022-11-15T09:15:41.734Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 440 2 - "-" "-" "c6df14c8-19ca-4f72-8477-a6042fd9a300" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:36723 10.116.3.8:15008 10.116.4.4:33730 - - inbound hcm
[2022-11-15T09:15:41.751Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 440 1 - "-" "-" "c07c267f-2907-4a93-98d6-396792afeeba" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:53265 10.116.3.8:15008 10.116.4.4:33730 - - inbound hcm
[2022-11-15T09:15:41.768Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "8d3fcd0e-f385-46fb-90b7-ecfa9438cb1f" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:43435 10.116.3.8:15008 10.116.4.4:33732 - - inbound hcm
[2022-11-15T09:15:41.785Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "7849ab1e-0a0d-48b3-9392-d75cbd02fff9" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:37219 10.116.3.8:15008 10.116.4.4:33732 - - inbound hcm
[2022-11-15T09:15:41.802Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "8a50ac87-c14d-4dc2-82de-f047394414ba" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:48129 10.116.3.8:15008 10.116.4.4:33732 - - inbound hcm
[2022-11-15T09:15:41.823Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 440 1 - "-" "-" "95227f1c-976a-478b-94f7-f24b42b40ea9" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:59963 10.116.3.8:15008 10.116.4.4:33732 - - inbound hcm
[2022-11-15T09:15:41.840Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 1 - "-" "-" "0eccee48-7246-4297-9024-80234de7934d" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:51511 10.116.3.8:15008 10.116.4.4:33730 - - inbound hcm
[2022-11-15T09:15:41.856Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 127 441 2 - "-" "-" "61a960f0-e958-4ed2-aa5d-b3e43cdc8366" "10.116.3.8:8080" "10.116.3.8:8080" virtual_inbound 10.116.4.4:43525 10.116.3.8:15008 10.116.4.4:33730 - - inbound hcm
[2022-11-15T09:15:41.823Z] "- - -" 0 - - - "-" 127 440 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60238 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.820Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 125 825 5 - "-" "-" "a13dd26a-8a4b-4a3d-b1d1-24764650d6c2" "10.116.4.9:8080" "10.116.4.9:8080" virtual_inbound 10.116.5.2:58637 10.116.4.9:15008 10.116.5.2:60216 - - inbound hcm
[2022-11-15T09:15:41.837Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 125 826 5 - "-" "-" "82475477-eada-428c-b4de-3299627e760b" "10.116.4.9:8080" "10.116.4.9:8080" virtual_inbound 10.116.5.2:57227 10.116.4.9:15008 10.116.5.2:60220 - - inbound hcm
[2022-11-15T09:15:41.839Z] "- - -" 0 - - - "-" 127 441 2 - "-" "-" "-" "-" "10.116.3.8:15008" outbound_tunnel_clus_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation 10.116.4.4:33730 10.116.3.8:8080 envoy://internal_client_address/ - - outbound tunnel
[2022-11-15T09:15:41.839Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60246 - - outbound capture listener
[2022-11-15T09:15:41.839Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60246 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.855Z] "- - -" 0 - - - "-" 127 441 2 - "-" "-" "-" "-" "10.116.3.8:15008" outbound_tunnel_clus_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation 10.116.4.4:33730 10.116.3.8:8080 envoy://internal_client_address/ - - outbound tunnel
[2022-11-15T09:15:41.855Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60256 - - outbound capture listener
[2022-11-15T09:15:41.855Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60256 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.853Z] "CONNECT - HTTP/2" 200 - via_upstream - "-" 125 826 6 - "-" "-" "885eb5ef-64f8-4e18-a519-9da9e859d827" "10.116.4.9:8080" "10.116.4.9:8080" virtual_inbound 10.116.5.2:33197 10.116.4.9:15008 10.116.5.2:60216 - - inbound hcm
chauhai1@QCV925JTJ7 ambient-mesh %
```

**You may also notice capture outbound (no waypoint proxy) in the log. Through the istiod control plane, ztunnel knows whether there is a waypoint proxy on the server side. We’ll explain this soon when we introduce some policies !**

```
kubectl -n istio-system logs -l app=ztunnel | grep '^\[' |grep waypoint
```
```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl -n istio-system logs -l app=ztunnel | grep '^\[' |grep waypoint
[2022-11-15T09:15:41.834Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41986 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.853Z] "- - -" 0 - - - "-" 125 826 7 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api/10.116.4.9:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/web-api_to_http_recommendation.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.201:8080 10.116.5.10:56824 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.850Z] "- - -" 0 - - - "-" 77 133 11 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep/10.116.5.10:8081" spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep_to_http_web-api.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.6.65:8080 10.116.5.9:41992 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.839Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60246 - - capture outbound (no waypoint proxy)
[2022-11-15T09:15:41.855Z] "- - -" 0 - - - "-" 127 441 3 - "-" "-" "-" "-" "envoy://outbound_tunnel_lis_spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation/10.116.3.8:8080" spiffe://cluster.local/ns/anz-ambient-demo/sa/recommendation_to_http_purchase-history.anz-ambient-demo.svc.cluster.local_outbound_internal envoy://internal_client_address/ 10.120.11.108:8080 10.116.4.9:60256 - - capture outbound (no waypoint proxy)
chauhai1@QCV925JTJ7 ambient-mesh 
```

### Important Point to notice ###

*What you've seen so far is that services can be added to the Mesh without restarting any Pod.*

*By adding services in the mesh, you get data encrypted automatically (between Kubernetes nodes) and observability at the L4 level.*

----
----
***
2. **L4 Authorization policies**


*While it is great to have mTLS among all my service connections, to fulfill a zero trust architecture, you’ll likely want to deny all accesses and only grant certain accesses explicitly. And now that services have been added to the mesh, you can easily apply authorization at the L4 level.*
| ------------------------------------------------------------------------------------------------------------------------- |

+ **For demo, purpose - we create a seperate namespace with name test-ambientmesh & deploy sleep pod to perform curl, from where we will test the connectivity as part of L4 Authorization policies**

```
chauhai1@QCV925JTJ7 ambient-mesh % kubens test-ambientmesh
Context "gke_anz-payments-lab-poc-5b24c4_australia-southeast1-a_ambient-mesh" modified.
Active namespace is "test-ambientmesh".

chauhai1@QCV925JTJ7 ambient-mesh % kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
sleep-67f77477d5-tf2fn   1/1     Running   0          31h
chauhai1@QCV925JTJ7 ambient-mesh % 

chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec -it sleep-67f77477d5-tf2fn sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # curl http://web-api.anz-ambient-demo.svc.cluster.local:8080/
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.116.5.10"
  ],
  "start_time": "2022-11-15T09:49:36.046566",
  "end_time": "2022-11-15T09:49:36.065442",
  "duration": "18.876541ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.116.4.9"
      ],
      "start_time": "2022-11-15T09:49:36.052317",
      "end_time": "2022-11-15T09:49:36.061732",
      "duration": "9.41476ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.116.3.8"
          ],
          "start_time": "2022-11-15T09:49:36.060150",
          "end_time": "2022-11-15T09:49:36.060278",
          "duration": "128.635µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
/ # 
```

+ **Deploy a deny all policy for everything in the anz-ambient-demo namespace:**
```
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: anz-ambient-demo
spec:
  {}
EOF
```
```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl get authorizationpolicy.security.istio.io/allow-nothing -n anz-ambient-demo
NAME            AGE
allow-nothing   59s
```
**You shouldn't be able to access the web-api application anymore.**

chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec -it sleep-67f77477d5-tf2fn sh                                         
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

% curl http://web-api.anz-ambient-demo.svc.cluster.local:8080/
curl: (52) Empty reply from server


### Now, let's create the different AuthorizationPolicy resources to only allow the communications needed. ###

```
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "web-api-rbac"
  namespace: anz-ambient-demo
spec:
  selector:
    matchLabels:
      app: web-api
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/anz-ambient-demo/sa/sleep"]
    - source:
        namespaces: ["test-ambientmesh"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "recommendation-rbac"
  namespace: anz-ambient-demo
spec:
  selector:
    matchLabels:
      app: recommendation
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/anz-ambient-demo/sa/web-api"]
    - source:
        namespaces: ["test-ambientmesh"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "purchase-history-rbac"
  namespace: anz-ambient-demo
spec:
  selector:
    matchLabels:
      app: purchase-history
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/anz-ambient-demo/sa/recommendation"]
    - source:
        namespaces: ["test-ambientmesh"]
EOF
```
```
Required authorizationpolicy got created
authorizationpolicy.security.istio.io/web-api-rbac created
authorizationpolicy.security.istio.io/recommendation-rbac created
authorizationpolicy.security.istio.io/purchase-history-rbac created
```
=======

You should now be able to access the web-api application again.

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec -it sleep-67f77477d5-tf2fn sh                                                                                  
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

# curl http://web-api.anz-ambient-demo.svc.cluster.local:8080/
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.116.5.10"
  ],
  "start_time": "2022-11-15T10:00:27.273261",
  "end_time": "2022-11-15T10:00:27.298603",
  "duration": "25.342577ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.116.4.9"
      ],
      "start_time": "2022-11-15T10:00:27.284757",
      "end_time": "2022-11-15T10:00:27.297132",
      "duration": "12.374893ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.116.3.8"
          ],
          "start_time": "2022-11-15T10:00:27.295505",
          "end_time": "2022-11-15T10:00:27.295629",
          "duration": "124.625µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

**Important point to notice**

```
But if we try to access it from an unauthorized Pod, access will be denied:

chauhai1@QCV925JTJ7 ambient-mesh % kubens anz-ambient-demo
Context "gke_anz-payments-lab-poc-5b24c4_australia-southeast1-a_ambient-mesh" modified.
Active namespace is "anz-ambient-demo".
chauhai1@QCV925JTJ7 ambient-mesh % 
chauhai1@QCV925JTJ7 ambient-mesh % 
chauhai1@QCV925JTJ7 ambient-mesh % anz-ambient-demo
chauhai1@QCV925JTJ7 ambient-mesh % 
chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec deploy/notsleep -- curl http://web-api:8080/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (52) Empty reply from server
command terminated with exit code 52
chauhai1@QCV925JTJ7 ambient-mesh % 
```
----
----
***

# Demo - Wavepoint proxy in action (Layer 7 features of istio) #

## List of features for demo:

1. L7 authorization policies
2. L7 Observability 
3. Traffic Shifting

### 1.  L7 authorization policies 

+ L4 policies are useful but may not be sufficient for our needs.

+ **For example, you’ll be able to send any request to the web-api service from the sleep pod while you may only want to allow requests with the GET method.**

+ **In order to have any L7 policy enforced for the web-api service, you’ll need to deploy a waypoint proxy for the service account used by the web-api pod.**

![image](https://user-images.githubusercontent.com/119640027/205218428-d7e0ee0e-5cae-40ad-8a98-d98f0f4bc547.png)



### Step 1

 *The Kubernetes gateway API is used to deploy a waypoint proxy for a given service account or the namespace.*

*The example below demonstrates how to ask Istio to deploy a waypoint proxy for the web-api service account.*


```
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: Gateway
metadata:
  name: web-api
  namespace: anz-ambient-demo
  annotations:
    istio.io/service-account: web-api
spec:
  gatewayClassName: istio-mesh
EOF
```

**Let's check that the waypoint proxy has been deployed correctly:**
```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl get pods -l ambient-type=waypoint
NAME                                    READY   STATUS    RESTARTS   AGE
web-api-waypoint-proxy-5bdf554b-jkthr   1/1     Running   0          6s
chauhai1@QCV925JTJ7 ambient-mesh %

NAME                                      READY   STATUS    RESTARTS   AGE
web-api-waypoint-proxy-7bbfdc784c-5vg9n   1/1     Running   0          69s
```

### Step 2

 *Now, let's allow the sleep pod to only send GET requests to the web-api service:*

```
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "web-api-rbac"
  namespace: anz-ambient-demo
spec:
  selector:
    matchLabels:
      app: web-api
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/anz-ambient-demo/sa/sleep"]
    - source:
        namespaces: ["test-ambientmesh"]
    to:
    - operation:
        methods: ["GET"]
EOF
```

### Step 3

 We should still be able to access the web-api application from the sleep pod.

kubectl exec deploy/sleep -- curl http://web-api:8080/

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec deploy/sleep -- curl http://web-api:8080/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1104  100  1104    0     0  10128 {    0 --:--:-- --:--:-- --:--:-- 10036
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.116.5.10"
  ],
  "start_time": "2022-11-15T10:54:50.352582",
  "end_time": "2022-11-15T10:54:50.419220",
  "duration": "66.63775ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.116.4.9"
      ],
      "start_time": "2022-11-15T10:54:50.380351",
      "end_time": "2022-11-15T10:54:50.396503",
      "duration": "16.151974ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.116.3.8"
          ],
          "start_time": "2022-11-15T10:54:50.393703",
          "end_time": "2022-11-15T10:54:50.393900",
          "duration": "197.192µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
     0 --:--:-- --:--:-- --:--:-- 10036
chauhai1@QCV925JTJ7 ambient-mesh % 
```
*From Another Namespace <test-ambientmesh> pod <sleep-67f77477d5-tf2fn>, which is whitelist to perform the action*

```
chauhai1@QCV925JTJ7 ambient-mesh % kubens test-ambientmesh ; kubectl exec deploy/sleep -- curl http://web-api.anz-ambient-demo.svc.cluster.local:8080/
Context "gke_anz-payments-lab-poc-5b24c4_australia-southeast1-a_ambient-mesh" modified.
Active namespace is "test-ambientmesh".
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1103  100  110{    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.116.5.10"
  ],
  "start_time": "2022-11-15T11:00:17.136142",
  "end_time": "2022-11-15T11:00:17.151681",
  "duration": "15.538742ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.116.4.9"
      ],
      "start_time": "2022-11-15T11:00:17.142233",
      "end_time": "2022-11-15T11:00:17.149134",
      "duration": "6.900293ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.116.3.8"
          ],
          "start_time": "2022-11-15T11:00:17.148085",
          "end_time": "2022-11-15T11:00:17.148197",
          "duration": "112.34µs",
        3  "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
    0     0  25651      0 --:--:-- --:--:-- --:--:-- 25651
chauhai1@QCV925JTJ7 ambient-mesh % 
```
### Step 4

 *But if you try to send a DELETE request, it will be denied:*

```
 chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec deploy/sleep -- curl http://web-api:8080/ -X DELETE
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    19  100    19    0     0    730      0 --:--:-- --:--:-- --:--:--   730
RBAC: access denied%      



chauhai1@QCV925JTJ7 ambient-mesh % kubens test-ambientmesh ; kubectl exec deploy/sleep -- curl http://web-api.anz-ambient-demo.svc.cluster.local:8080/ -X DELETE
Context "gke_anz-payments-lab-poc-5b24c4_australia-southeast1-a_ambient-mesh" modified.
Active namespace is "test-ambientmesh".
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    19  100    19    0     0    441      0 --:--:-- --:--:-- --:--:--   441RBAC: access denied
chauhai1@QCV925JTJ7 ambient-mesh % 
```

----
----
***

### 2. L7 Observability ##



**With waypoint proxy deployed for the web-api service, you automatically get L7 metrics for this service.**

*For example, you can view the 403 response code from the web-api service’s waypoint proxy’s /stats/prometheus endpoint:*

kubectl exec deploy/web-api-waypoint-proxy -- curl http://localhost:15020/stats/prometheus | grep istio_requests_total 

```
# kubectl exec deploy/web-api-waypoint-proxy -- curl http://localhost:15020/stats/prometheus | grep istio_requests_total

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0# TYPE istio_requests_total counter
istio_requests_total{response_code="200",reporter="destination",source_workload="sleep",source_workload_namespace="default",source_principal="spiffe://cluster.local/ns/default/sa/sleep",source_app="unknown",source_version="unknown",source_cluster="Kubernetes",destination_workload="web-api",destination_workload_namespace="default",destination_principal="spiffe://cluster.local/ns/default/sa/web-api",destination_app="unknown",destination_version="unknown",destination_service="web-api.default.svc.cluster.local",destination_service_name="web-api",destination_service_namespace="default",destination_cluster="Kubernetes",request_protocol="http",response_flags="-",grpc_response_status="",connection_security_policy="mutual_tls",source_canonical_service="sleep",destination_canonical_service="web-api",source_canonical_revision="latest",destination_canonical_revision="v1"} 1


istio_requests_total{response_code="403",reporter="destination",source_workload="sleep",source_workload_namespace="default",source_principal="spiffe://cluster.local/ns/default/sa/sleep",source_app="unknown",source_version="unknown",source_cluster="Kubernetes",destination_workload="web-api",destination_workload_namespace="default",destination_principal="spiffe://cluster.local/ns/default/sa/web-api",destination_app="unknown",destination_version="unknown",destination_service="web-api.default.svc.cluster.local",destination_service_name="web-api",destination_service_namespace="default",destination_cluster="Kubernetes",request_protocol="http",response_flags="-",grpc_response_status="",connection_security_policy="mutual_tls",source_canonical_service="sleep",destination_canonical_service="web-api",source_canonical_revision="latest",destination_canonical_revision="v1"} 1


istio_requests_total{response_code="200",reporter="source",source_workload="web-api-waypoint-proxy",source_workload_namespace="default",source_principal="unknown",source_app="unknown",source_version="unknown",source_cluster="Kubernetes",destination_workload="recommendation",destination_workload_namespace="default",destination_principal="unknown",destination_app="unknown",destination_version="unknown",destination_service="recommendation.default.svc.cluster.local",destination_service_name="recommendation",destination_service_namespace="default",destination_cluster="Kubernetes",request_protocol="http",response_flags="-",grpc_response_status="",connection_security_policy="unknown",source_canonical_service="web-api-waypoint-proxy",destination_canonical_service="recommendation",source_canonical_revision="latest",destination_canonical_revision="v1"} 1
100  220k    0  220k    0     0  31.2M      0 --:--:-- --:--:-- --:--:-- 35.8M
root@virtualmachine:~# 
```


### 3. Traffic Shifting ###

**Canary Deployment**

+ A canary test is often performed to ensure the new version of a service not only functions properly but also doesn’t cause a degradation in performance or reliability.


**For example, if there is a new version (v2) of the purchase-history service, you may want to gradually roll out the v2 version of the service by sending only 10% of the traffic to it.**

```
Deploy the v2 version of purchase-history service:

kubectl apply -f files/L7-features-wave_point_proxy/Traffic_shifting/canary_purchase_history-v2.yaml 
```
**Important Action**

### Step -1

*In order to apply traffic shift on the purchase-history service, you’ll need to deploy a waypoint proxy for the service, similar to deploying a waypoint proxy via a Kubernetes gateway resource for the web-api service earlier.*

```
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: Gateway
metadata:
  name: purchase-history
  annotations:
    istio.io/service-account: purchase-history
spec:
  gatewayClassName: istio-mesh
EOF
```
### Step 2

Apply a VirtualService resource that configures the traffic shifting:

```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: purchase-history-vs
spec:
  hosts:
  - purchase-history.default.svc.cluster.local
  http:
  - route:
    - destination:
        host: purchase-history.default.svc.cluster.local
        subset: v1
        port:
          number: 8080
      weight: 90
    - destination:
        host: purchase-history.default.svc.cluster.local
        subset: v2
        port:
          number: 8080
      weight: 10
EOF
```
### Step 3

The subset referred in the above VirtualService must be defined in a DestinationRule resource so that Istio knows which subsets map to what labels for the purchase-history host.

```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: purchase-history-dr
spec:
  host: purchase-history.default.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
EOF
```

*You can confirm that around 10% of the traffic is sent to the v2 version of the purchase-history service by running the following command:*

```
kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s http://web-api:8080/; done | grep -c purchase-history-v2'

 kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s http://web-api:8080/; done | grep -c purchase-history-v2'
8
root@virtualmachine:~# kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s http://web-api:8080/; done | grep -c purchase-history-v2'
10
root@virtualmachine:~# kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s http://web-api:8080/; done | grep -c purchase-history-v2'
7
root@virtualmachine:~# kubectl exec deploy/sleep -- sh -c 'for i in $(seq 1 100); do curl -s http://web-api:8080/; done | grep -c purchase-history-v2'
8

This command has sent a total of 100 requests, so you should get around 10 requests sent to v2.

```

# Demo - Ambient Interoperability with sidecars #


+ All the Pods don't need to use the new Ambient mode.

+ You can have some Pods using sidecars while others are using Ambient.

### Step 1

Let's create a new namespace called httpbin:

kubectl create namespace httpbin

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl create namespace httpbin
namespace/httpbin created

chauhai1@QCV925JTJ7 ambient-mesh % kubectl describe ns httpbin
Name:         httpbin
Labels:       kubernetes.io/metadata.name=httpbin
Annotations:  <none>
```
### Step 2

*To use sidecars in this namespace, you need to label it accordingly:*

kubectl label namespace httpbin istio-injection=enabled

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl label namespace httpbin istio-injection=enabled
namespace/httpbin labeled
chauhai1@QCV925JTJ7 ambient-mesh % kubectl describe ns httpbin                            
Name:         httpbin
Labels:       istio-injection=enabled
              kubernetes.io/metadata.name=httpbin
Annotations:  <none>
Status:       Active
```
### Step 3

Then, you can deploy the httpbin application:

chauhai1@QCV925JTJ7 ambient-mesh % kubens httpbin 


```
kubectl apply -n httpbin -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: httpbin
---
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
    service: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 80
  selector:
    app: httpbin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      serviceAccountName: httpbin
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        ports:
        - containerPort: 80
EOF
```
```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl get po
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-74fb669cc6-n6n2s   2/2     Running   0          54s
chauhai1@QCV925JTJ7 ambient-mesh % 
```

### Step 4

*Finally, we can send a request from the sleep Pod (ambient mode) to the httpbin Pod (sidecar):*

 kubectl exec deploy/sleep -n anz-ambient-demo  -- curl http://httpbin.httpbin.svc.cluster.local:8000/get

```
chauhai1@QCV925JTJ7 ambient-mesh % kubectl exec deploy/sleep -n anz-ambient-demo  -- curl http://httpbin.httpbin.svc.cluster.local:8000/get 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   582  100   582    0     0   5388      0 --:--:-- --:--:-- --:--:--  5388
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.httpbin.svc.cluster.local:8000", 
    "User-Agent": "curl/7.69.1", 
    "X-B3-Sampled": "0", 
    "X-B3-Spanid": "ebbb838f63173605", 
    "X-B3-Traceid": "618bfc48907765f9ebbb838f63173605", 
    "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/httpbin/sa/httpbin;Hash=bda293a02a7692b9f3f02b837566d7c2c7c8021206315c205ca46246dbe6d871;Subject=\"\";URI=spiffe://cluster.local/ns/anz-ambient-demo/sa/sleep"
  }, 
  "origin": "127.0.0.1", 
  "url": "https://httpbin.httpbin.svc.cluster.local:8000/get"
}
chauhai1@QCV925JTJ7 ambient-mesh % 
```
### Outcome

You can see that the httpbin application has received the request with the X-Forwarded-Client-Cert indicating that the request was sent by a Pod with the identity corresponding to the sleep service account.

![image](https://user-images.githubusercontent.com/119640027/205218325-27c7f97f-0021-48f7-9bc7-6791f26026da.png)


## End of Demo 

