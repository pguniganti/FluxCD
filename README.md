# FluxCD
### Installing and Configuring Prerequisites

1. **Install Helm**:
   - Download Helm.
   - Add Helm to your system’s PATH.
   - Verify installation.

2. **Install kubectl**:
   - Download kubectl.
   - Add kubectl to your system’s PATH.
   - Verify installation.

### Deploying FluxCD

3. **Install FluxCD**:
   ```sh
   kubectl apply -f https://github.com/fluxcd/flux2/releases/latest/download/install.yaml

4. **Bootstrap FluxCD**:
flux bootstrap github \
  --owner=<your-github-username> \
  --repository=<your-repo-name> \
  --branch=main \
  --path=clusters/my-cluster
```sh
5. **Add Prometheus Helm Repository**:
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```sh
6. **Create HelmRepository Resource:**
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: prometheus-community
  namespace: flux-system
spec:
  interval: 10m
  url: https://prometheus-community.github.io/helm-charts
```sh
7. **Apply HelmRepository:**
kubectl apply -f helm-repository.yaml

8. **Create HelmRelease Resource:**
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: kube-prometheus-stack
  namespace: monitoring
spec:
  interval: 5m
  timeout: 10m
  chart:
    spec:
      chart: kube-prometheus-stack
      sourceRef:
        kind: HelmRepository
        name: prometheus-community
        namespace: flux-system
  values:
    prometheus:
      serviceMonitor:
        selfMonitor: true
```sh

9. **Apply HelmRelease:**
kubectl apply -f helm-release.yaml
```sh

10. **Reconcile HelmRelease:**
flux reconcile helmrelease kube-prometheus-stack -n monitoring
```sh

11. **Verify Services:**
kubectl get pods -n monitoring
```sh

12. **Install Nginx Ingress Controller:**
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true -n ingress-nginx --create-namespace
```sh

13. **Create Ingress Resources**
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
spec:
  rules:
  - host: grafana.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kube-prometheus-stack-grafana
            port:
              number: 80
``sh

14. **Create Prometheus Ingress:**
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: monitoring
spec:
  rules:
  - host: prometheus.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kube-prometheus-stack-prometheus
            port:
              number: 9090
``sh

15. **Apply Ingress Resources:**
kubectl apply -f grafana-ingress.yaml
kubectl apply -f prometheus-ingress.yaml
``sh

16. **Access Services:**
Grafana: http://a048b014ec0084b97923be0c98cdd0be-1015424521.us-east-1.elb.amazonaws.com

Prometheus: Check your LoadBalancer DNS.






