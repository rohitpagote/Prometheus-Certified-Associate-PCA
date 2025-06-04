# File Service Discovery
- File Service Discovery is a mechanism to import jobs and target configurations into Prometheus from an **external file**.
- File service discovery supports both `JSON` and `YAML` file formats.
- This approach provides the flexibility to use any file containing valid configuration data, and can even leverage glob patterns (e.g., `*.json`) to include multiple files at once.

## Configuring File Service Discovery in Prometheus
- To enable file service discovery, add a `file_sd_configs` block to the Prometheus configuration file (typically named `prometheus.yaml`). 
- Within this block, specify the file or files holding the configuration details. 
- For example, `targets.json`, `*.json`

### Example:

- **JSON configuration file containing targets:**
    - Below is an example configuration file in JSON format defining three job setups:

```json
[
  {
    "targets": ["node1:9100", "node2:9100"],
    "labels": {
      "team": "dev",
      "job": "node"
    }
  },
  {
    "targets": ["localhost:9090"],
    "labels": {
      "team": "monitoring",
      "job": "prometheus"
    }
  },
  {
    "targets": ["db1:9090"],
    "labels": {
      "team": "db",
      "job": "database"
    }
  }
]
```

- **Prometheus YAML Configuration (`prometheus.yaml`):**
    - Here is how you can reference the JSON configuration in your Prometheus YAML file:

```yml
scrape_configs:
  - job_name: file-example
    file_sd_configs:
      - files:
          - 'file-sd.json'
          - '*.json'
```

- In this setup, Prometheus imports labels and targets defined in the JSON file. For instance:
    - A job with the label job: node targets specific nodes.
    - Another job is dedicated to Prometheus monitoring.
    - A third job is defined for database monitoring.

After configuring file service discovery in your `prometheus.yaml`, restart Prometheus. On restart, we should see all endpoints appear as if they were defined directly in the configuration file.
