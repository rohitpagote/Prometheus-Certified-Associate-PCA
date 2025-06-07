# Introduction - Monitoring Kubernetes
Using Prometheus we can monitor 2 things in Kubernetes:
1. Applications running in a Kubernetes environment
- These include workloads such as web applications, web servers, and other services deployed in the cluster.
2. Kubernetes cluster itself
- Control-Plane components (API Server, coredns, kube-scheduler)
- Kubelet (cAdvisor) - exposing container metrics
- Kube-state-metrics - cluster level metrics (deployment, pods metrics)
- Node-exporter - runs on all nodes for host related metrics (cpu, memory, network, etc)

> [!NOTE]
> By default, Kubernetes does not expose cluster-level metrics for pods, deployments, or services. To collect these metrics, we must deploy the kube-state metrics container, which then advertises critical information directly to Prometheus.

---

# Deploying Node Exporter with DaemonSets
- We can manually install Node Exporter on each node or include it in the node images but a more efficient solution in Kubernetes is to deploy **Node Exporter using a DaemonSet**. 
- This approach ensures that every node, including any new nodes joining the cluster, will automatically run the Node Exporter pod.

- Kubernetes service discovery utilizes the cluster API to automatically detect all endpoints that need to be scraped by Prometheus. 
- This automatic discovery covers base Kubernetes components, Node Exporters running on every node, and the kube-state metrics container, eliminating the need for manual endpoint configurations.

---

# Deploying Prometheus on Kubernetes
There are two main strategies for deploying Prometheus on a Kubernetes cluster:
 
## 1. Manual Deployment
- This method involves manually creating various Kubernetes objects such as Deployments, Services, ConfigMaps, and Secrets. 
- While this approach provides granular control, it requires extensive configuration and is generally more complex.

## 2. Using Helm and the Prometheus Operator
- A more straightforward approach involves deploying Prometheus using a Helm chart, specifically the **kube-prometheus-stack** from the Prometheus Community repository. 
- Helm - a package manager for Kubernetes - bundles all necessary configurations and dependencies into a single package, making deployment and management significantly easier.

---

# The Prometheus Operator and Custom Resources
- The Prometheus Operator simplifies the management of Prometheus within a Kubernetes environment. 
- By extending the Kubernetes API with custom resources, the operator streamlines tasks such as initialization, configuration, scaling, upgrading, and lifecycle management of complex applications.
- Using the Prometheus Operator, we can deploy a Prometheus instance via a custom resource named **Prometheus** rather than managing Deployments or StatefulSets manually. 
- This abstraction simplifies modifications and ongoing operations.
- For example, a Prometheus custom resource might be defined as follows:

```yml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus
    meta.helm.sh/release-namespace: default
  labels:
    app: kube-prometheus-stack-prometheus
    name: prometheus-kube-prometheus-prometheus
spec:
  alerting:
    alertmanagers:
      - apiVersion: v2
        name: prometheus-kube-prometheus-alertmanager
        namespace: default
        pathPrefix: /
        port: http-web
```

- The operator also provides additional custom resources to manage Alertmanager configurations, Prometheus rules, ServiceMonitor, and PodMonitor resources. 
- These abstractions allow to declare and adjust Prometheus components without directly manipulating low-level Kubernetes objects.
