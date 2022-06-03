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
  
  Delete ValidatingWebhookConfiguration
  
  ```shell
  kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
  ```
