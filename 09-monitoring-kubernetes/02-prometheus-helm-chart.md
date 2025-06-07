# Installing Helm
- Before installing the Prometheus chart, we need to have Helm installed on the system. 
- For detailed installation instructions specific to your operating system, refer to the official [Helm Documentation](https://helm.sh/docs/intro/install/). 
- For a standard Linux machine, we can use the built-in installer script.

1. Download the Helm installer script:
```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

2. Change the script’s permissions to make it executable and run the installer:
```bash
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

3. Verify the installation by checking the Helm version:
```bash
$ helm version
version.BuildInfo{Version:"v3.10.2", GitCommit:"50f003e5ee8704ec937a756c646870227d7c8b58", GitTreeState:"clean", GoVersion:"go1.18.8"}
```

---

# Adding the Prometheus Repository and Installing the Chart
- With Helm installed, we can now add the Prometheus Community repository and deploy the kube-prometheus-stack chart.

1. Add the Prometheus Community repository:
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

2. Update Helm repositories:
```bash
$ helm repo update
```

3. Install the **kube-prometheus-stack** chart with your chosen release name (in this example, we use `prometheus`):
```bash
$ helm install prometheus prometheus-community/kube-prometheus-stack
```

---

# Customizing the Chart Values
- Each Helm chart allows to override default configuration values. 
- To explore configurable options for the kube-prometheus-stack chart, export the default configuration into a file:
```bash
$ helm show values prometheus-community/kube-prometheus-stack > values.yaml
```

- Review the generated `values.yaml` file to see all the custom configuration options. 
- Below is an excerpt (a small piece of code) that illustrates some of the configurable sections:
```yml
containers: []

# Additional volumes on the output StatefulSet definition.
volumes: []

# Additional VolumeMounts on the output StatefulSet definition.
volumeMounts: []

## InitContainers allows injecting additional initContainers. This is meant to allow doing some changes
## (permissions, directory tree modifications) on mounted volumes before starting Prometheus.
initContainers: []

## Priority class assigned to the Pods.
priorityClassName: ""

## PortName to use for ThanosRuler.
portName: "web"

## ExtraSecret can be used to store various data in an extra secret (e.g., hashed basic auth credentials).
extraSecret:
  name: ""
  annotations: {}
  data: {}

## Setting to true produces cleaner resource names, but requires a data migration due to persistent volume name changes.
cleanPrometheusOperatorObjectNames: false
```

- Once we have customized the `values.yaml` file, deploy your configuration using the `-f` flag:
```bash
$ helm install prometheus prometheus-community/kube-prometheus-stack -f values.yaml
```
