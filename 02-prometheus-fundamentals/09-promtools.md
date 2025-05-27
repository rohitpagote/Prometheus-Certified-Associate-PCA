# Introduction to PromTools
PromTools is a powerful utility that comes bundled with Prometheus. PromTools offers a range of functions that include:
- Validating configurations files (e.g. `prometheus.yml` and rule files)
- Verifying metric formatting for Prometheus consumption
- Executing ad hoc queries on the Prometheus server
- Debugging and profiling
- Unit testing recording and alerting rules

## Validating Prometheus Configuration
To verify or validate the Prometheus configuration file, we have `check config` command provided by PromTools.
This ensures that the setup is error-free before deployment, preventing potential downtime.

For example, to validate your configuration, run:
```bash
$ promtool check config /etc/prometheus/prometheus.yml
Checking prometheus.yml
SUCCESS: prometheus.yml is valid prometheus config file syntax
```

## Handling Configuration Errors
Consider a scenario where the configuration file `prometheus.yml` contains a typo. 
```yml
scrape_configs:
  - job_name: "node"
    metric_path: "/metrics"
    static_configs:
      - targets: ["node1:9100"]

# Should be “metrics_path”
```

For instance, we have mistakenly use "metric_path" instead of the correct "metrics_path," PromTools will catch the error when you run the check again. The output may look like this:
```bash
$ promtool check config /etc/prometheus/prometheus.yml
Checking prometheus.yml
FAILED: parsing YAML file prometheus.yml: yaml: unmarshal errors:
  line 24: field metric_path not found in type config.ScrapeConfig
```

> [!WARNING]
> Never deploy configuration changes without first validating them using PromTools. A misconfigured file could prevent your Prometheus server from starting, leading to potential downtime.