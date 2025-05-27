# Prometheus in Docker Container
We can run Prometheus inside a Docker container, making it easier to manage in a containerized environment.

## Step 1: Configure Prometheus
Create and edit the Prometheus configuration file `prometheus.yml` just similar to a bare-metal linux machine. The configuration remains identical.

```yml
global:
  scrape_configs:
    - job_name: "prometheus"
      static_configs:
        - targets: ["localhost:9090"]
```

## Step 2: Run Prometheus in a Docker Container
Next, pull the official Prometheus image from Docker Hub and run it. Expose the default port 9090 and bind mount the configuration file to ensure that changes made on the host are immediately reflected in the container.

```bash
$ vi prometheus.yml
$ docker run -d -v /path-to/prometheus.yml:/etc/prometheus/prometheus.yml -p 9090:9090 prom/prometheus
```

**Command Breakdown**:
- `vi prometheus.yml`: Opens the configuration file for editing.
- `docker run -d`: Runs the docker container in detached mode.
- `-v`: Creates a bind mount, mapping the local configuration file to the container’s `/etc/prometheus/prometheus.yml` path.
- `-p 9090:9090`: Maps the container's port 9090 to the host, allowing to access Prometheus.

With these steps completed, the Prometheus instance running inside a Docker container will listen on port 9090. Any modifications to the `prometheus.yml` file on host will be automatically applied within the container.
