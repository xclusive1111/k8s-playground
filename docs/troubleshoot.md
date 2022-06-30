- Containerd configuration issues:
  [Impossible to create or start a container after reboot (OCI runtime create failed](https://github.com/containerd/containerd/issues/4857#issuecomment-747238907)

- Pod stuck at ContainerCreating:
  
  ```shell
  kubectl describe pods my-pod
  ```

- Kubernetes namespaces stuck in terminating statuskube
  
  ```shell
  NAMESPACE=<my-namespace>
  kubectl get namespace $NAMESPACE -o json > $NAMESPACE.json
  sed -i -e 's/"kubernetes"//' $NAMESPACE.json
  kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f ./$NAMESPACE.json
  ```

- Ingress nginx controller fail to create ingress resource due to admission webhook failture:
  
  ```
  Error from server (InternalError): error when creating "deployments/web1.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": context deadline exceeded
  ```
  
  Try to turn on an admission controller: [Using Admission Controllers | Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#how-do-i-turn-on-an-admission-controller) or delete `ValidatingWebhookConfiguration`
  
  ```shell
  kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
  ```
* The `calico-apiserver` deployment needs to run with `hostNetwork: true` and `dnsPolicy: ClusterFirstWithHostNet`, otherwise the `kube-apiserver` will fail to validate apiservice `v3.projectcalico.org`

* Kubernetes dashboard:
  
  * The `metrics-server` read metrics of pods, deployments and nodes
  
  * The `metrics-scraper` read metrics from the `metrics-server`
  
  * The `dashboard` provides UI administration, and read metrics from its side-car  client, which is the `metrics-scraper` (need to set as argument)
  
  * In order for the dashboard to work, the `kube-apiserver` must  [enable an aggregation layer](https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/) by setting `--enable-aggregator-routing=true` option, and provide proper flags to validate client authentication. See [Configure Apiserver Client Authentication](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-aggregation-layer/#kubernetes-apiserver-client-authentication)

* Config custom registry
  
  * On each worker nodes, modify containerd config to add this entry:
    
    ```shell
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."myregistry.com"]
      endpoint = ["https://myregistry.com"]
    [plugins."io.containerd.grpc.v1.cri".registry.configs."myregistry.com".tls]
      ca_file = "/usr/local/share/ca-certificates/myregistry.com.crt"
    ```
