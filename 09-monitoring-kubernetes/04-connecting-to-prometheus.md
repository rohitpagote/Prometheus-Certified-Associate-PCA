# Listing Cluster Services
List all services in your Kubernetes cluster to locate the Prometheus service using below command:
```bash
$ kubectl get service

NAME                                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
service/alertmanager-operated                       ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   158m
service/kubernetes                                  ClusterIP   10.100.0.1       <none>        443/TCP                      3d19h
service/prometheus-grafana                          ClusterIP   10.100.235.247   <none>        80/TCP                       158m
service/prometheus-kube-prometheus-alertmanager     ClusterIP   10.100.253.114   <none>        443/TCP                      158m
service/prometheus-kube-prometheus                  ClusterIP   10.100.133.149   <none>        8080/TCP                     158m
service/prometheus-kube-state-metrics               ClusterIP   None             <none>        9090/TCP                     158m
service/prometheus-operated                         ClusterIP   10.100.248.61    <none>        9100/TCP                     158m
service/prometheus-prometheus-node-exporter         ClusterIP   None             <none>        9100/TCP                     158m
```

> [!NOTE]
> The service is of type `ClusterIP`, meaning it is accessible only within the cluster.

---

# Inspecting the Prometheus Service Configuration
- To investigate further, output the service configuration to a YAML file using below command:
```bash
kubectl get service prometheus-kube-prometheus-prometheus -o yaml > service.yaml

apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus
    meta.helm.sh/release-namespace: default
    creationTimestamp: "2022-11-21T18:23:09Z"
  labels:
    app: kube-prometheus-stack-prometheus
    app.kubernetes.io/instance: prometheus
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/part-of: kube-prometheus-stack
    app.kubernetes.io/version: 41.9.1
    chart: kube-prometheus-stack-41.9.1
    heritage: Helm
resourceVersion: "801692"
spec:
  clusterIP: 10.100.54.169
  clusterIPs:
    - 10.100.54.169
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: http-web
      port: 9090
      protocol: TCP
      targetPort: 9090
  selector:
    app.kubernetes.io/name: prometheus
    prometheus: prometheus-kube-prometheus-prometheus
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

---

# Enabling External Access
- To access Prometheus externally, consider the following options:
    - Change the service type to `NodePort` or `LoadBalancer`.
    - Set up an `Ingress` resource to route traffic from a specific URL or domain to the Prometheus service.
- For demo purposes, we'll use port forwarding.

---

# Accessing Prometheus via Port Forwarding
- First, check the running pods in the cluster:
```bash
kubectl get pod
```
- After identifying the correct Prometheus pod (for example, `prometheus-prometheus-kube-prometheus-0`), forward the pod’s port `9090` to the local machine:
```bash
kubectl port-forward prometheus-prometheus-kube-prometheus-0 9090
```
- With port forwarding active, open the browser and navigate to `http://localhost:9090` to access the Prometheus web UI.
