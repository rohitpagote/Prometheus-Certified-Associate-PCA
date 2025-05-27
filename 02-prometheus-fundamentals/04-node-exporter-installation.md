# Monitor Linux Server Using Prometheus Node Exporter
## Step 1: Copy URL of the Node Exporter.
1. Go to official Prometheus [downloads page](https://prometheus.io/download/). Copy URL of the [node_exporter](https://prometheus.io/download/#node_exporter) for Linux "tar" file.

![Node Exporter Download Page](images/node_exporter_download_page.png)

2. Run the following command to download package. Paste the copied URL after wget in the below command:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
```

3. Now go to Node Exporter downloaded location and extract it using `tar` command.
```bash
tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz
```

4. Rename it as per your preference (optional).
```bash
mv node_exporter-1.9.1.linux-amd64.tar.gz node-exporter-package
```

## Step 2: Configure Prometheus
1. Create a user for the node exporter.
```bash
useradd --no-create-home --shell /bin/false nodeuser
```
- As we do not need `nodeuser` user to login and have home directory.

2. Move binary to “/usr/local/bin” from the downloaded extracted package.
Copy `node_exporter` binary from the `node-exporter-package` folder to `/usr/local/bin`.
```bash
cp  node-exporter-package/node_exporter /usr/local/bin
```

3. Change the ownership to `nodeuser` user.
```bash
chown nodeuser:nodeuser usr/local/bin/node_exporter
```

## Step 3: Configure Node Exporter Service Unit file
1. Create the Node Exporter Service File.
```bash
vi /etc/systemd/system/node_exporter.service
```

2. Add the following content to the file.
```bash
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeuser
Group=nodeuser
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

3. Reload the systemd service.
```bash
systemctl daemon-reload
```

4. Start the node_exporter service.
```bash
systemctl start node_exporter
```

5. Check the node_exporter service status (optional)
```bash
systemctl status node_exporter
```

6. Enable the node_exporter service so that it can get automatically started after boot
```bash
systemctl enable node_exporter
```

## Step 4: View the metrics browsing node exporter URL
- Use the following URL to view the systems default metrics.
```bash
http://IP-Address:9100/metrics     # replace IP-Address with the IP of you host (localhost) or VM
```
- You will see the following interface.

![Node Exporter Metrics](images/node_exporter_metrics.png)

---

> [!NOTE]
> *Follow below steps to configure/register Node Exporter target (machine) on Prometheus server.*

## Step 5: Configure Node Exporter Target on Prometheus Server
1. Login to Prometheus server and modify the `prometheus.yml` file.
```bash
vi /etc/prometheus/prometheus.yml
```

2. Add the following configurations under the scrape config.
```yaml
- job_name: 'node_exporter_job'
    scrape_interval: 5s
    static_configs:
      - targets: ['<target-ip-address>:9100']
```

3. Restart the Prometheus Service.
```bash
systemctl restart prometheus
```

4. Login to Prometheus server web interface, and check targets.
```bash
http://<prometheus-server-IP>:9090/targets     # replace prometheus-server-IP with the IP of you host (localhost) or VM
```