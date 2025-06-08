# Prometheus Chart Overview
- List every resource created by the Helm chart using below command:
```bash
kubectl get all
```
- The output displays several components.

---

## StatefulSets
There are two StatefulSets in the cluster:
1. **Prometheus StatefulSet**
    - This StatefulSet creates the Prometheus server instance. 
    - Although the name may be long, it represents the actual Prometheus instance. 
2. **Alertmanager StatefulSet** 
    - This StatefulSet is responsible for running Alertmanager, which handles alert notifications.

---

## Deployments
Key deployments include:
1. **Prometheus Grafana Deployment**
    - Grafana serves as the graphical UI tool to help visualize data from Prometheus. 
    - It is automatically deployed and configured via the Helm chart.
2. **Kube Prometheus Operator Deployment**
    - The Prometheus Operator manages the lifecycle of the Prometheus instance, including configuration updates and restarts as needed.
3. **Kube-state-metrics Deployment**
    - This deployment runs a container that gathers metrics about Kubernetes objects (for example, deployments, services, and pods).

- ReplicaSets corresponding to these deployments are also present and ensure that the correct number of pod replicas is maintained.

---

## DaemonSet
- There is a DaemonSet called Node Exporter. 
- This resource deploys a Node Exporter Pod on every cluster node, including any nodes added later. 
- The Node Exporter collects host-level metrics such as CPU utilization, memory usage, and file system details. 

---

## Pods and Services
- The Pods section lists all deployed pods, including:
    - Prometheus server pod
    - Alertmanager pod
    - Grafana pod
    - Prometheus Operator pod
    - kube-state-metrics pod
    - Two Node Exporter pods (one per node)

- The Services section exposes these pods as ClusterIP services, meaning they are accessible only within the cluster. 
- To expose the Prometheus server or Grafana externally, we would need to configure an ingress, load balancer, or proxy.

---

Below is an excerpt from the output of kubectl get all:
```bash
NAME                                                           READY   STATUS    RESTARTS   AGE
pod/alertmanager-prometheus-kube-prometheus-alertmanager-0      2/2     Running   1          158m
pod/prometheus-grafana-d978ddcb9-xhvgw                          3/3     Running   0          158m
pod/prometheus-kube-state-metrics-649f8795d4-fltnq              1/1     Running   0          158m
pod/prometheus-prometheus-prometheus-0                          1/1     Running   0          158m
pod/prometheus-node-exporter-xntb2                              1/1     Running   0          158m


NAME                                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
service/alertmanager-operated                       ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   158m
service/kubernetes                                  ClusterIP   10.100.0.1       <none>        443/TCP                      3d19h
service/prometheus-grafana                          ClusterIP   10.100.235.247   <none>        80/TCP                       158m
service/prometheus-kube-prometheus-alertmanager     ClusterIP   10.100.253.114   <none>        443/TCP                      158m
service/prometheus-kube-prometheus                  ClusterIP   10.100.133.149   <none>        8080/TCP                     158m
service/prometheus-kube-state-metrics               ClusterIP   None             <none>        9090/TCP                     158m
service/prometheus-operated                         ClusterIP   10.100.248.61    <none>        9100/TCP                     158m
service/prometheus-prometheus-node-exporter         ClusterIP   None             <none>        9100/TCP                     158m


NAME                                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/prometheus-prometheus-node-exporter          2         2         2       2            2           <none>          158m


NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-grafana                  1/1     1           1           158m
deployment.apps/prometheus-kube-prometheus-operator 1/1     1           1           158m
deployment.apps/prometheus-kube-state-metrics       1/1     1           1           158m


NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-grafana-d978ddcb9                  1         1         1       158m
replicaset.apps/prometheus-kube-prometheus-operator-66c5f68b9 1         1         1       158m
replicaset.apps/prometheus-kube-state-metrics-649f8795d4      1         1         1       158m


NAME                                          READY   AGE
statefulset.apps/alertmanager-prometheus-kube-prometheus-alertmanager   1/1    158m
statefulset.apps/prometheus-prometheus                                  1/1    158m
```
