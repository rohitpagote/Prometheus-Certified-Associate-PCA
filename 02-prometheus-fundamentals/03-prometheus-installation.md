# Install and Configure prometheus on Ubuntu Linux
## Step 1: Update the system
```bash
sudo apt update -y
```

## Step 2: Download and untar Prometheus package
1. Go to official Prometheus [downloads page](https://prometheus.io/download/), and copy the URL of Linux "tar" file.

![Prometheus Download Page](images/prometheus-download-page.png)

2. Run the following command to download package. Paste the copied URL after wget in the below command:
```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.4.0-rc.0/prometheus-3.4.0-rc.0.linux-amd64.tar.gz
```

3. Now go to Prometheus downloaded location and extract it using `tar` command.
```bash
tar -xvf prometheus-3.4.0-rc.0.linux-amd64.tar.gz
```

4. Rename it as per your preference (optional).
```bash
mv prometheus-3.4.0-rc.0.linux-amd64 prometheus-package
```

## Step 3: Configure Prometheus
1. Create a `prometheus` user.
```bash
useradd --no-create-home --shell /bin/false prometheus
```
- As we do not need `prometheus` user to login and have home directory.

2. Create needed directories.
```bash
mkdir /etc/prometheus
mkdir /var/lib/prometheus
```

3. Change the owner of the above directories.
```bash
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus
```

4. Copy `prometheus` and `promtool` binary from the `prometheus-package` folder to `/usr/local/bin`.
```bash
cp  prometheus-package/prometheus /usr/local/bin
cp prometheus-package/promtool /usr/local/bin
```

5. Change the ownership to `prometheus` user.
```bash
chown prometheus:prometheus usr/local/bin/prometheus
chown prometheus:prometheus usr/local/bin/promtool
```

*Below sub-steps (6 and 7) are optional, follow only if the `prometheus-package` contains `console` and `console-libraries` directories.*

6. Copy `consoles` and `console_libraries` directories from the `prometheuspackage` to `/etc/prometheus folder`
```bash
cp -r prometheus-package/consoles /etc/prometheus
cp -r prometheus-package/console-libraries /etc/prometheus
```

7. Change the ownership to `prometheus` user
```bash
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

8. Copy `prometheus.yml` file from the `prometheus-package` folder to `/etc/prometheus/`.
```bash
cp prometheus-package/prometheus.yml /etc/prometheus/
```

9. Change the ownership of the file.
```bash
chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

## Step 4: Configure Prometheus Service Unit file
1. Create the Prometheus Service File.
```bash
vi /etc/systemd/system/prometheus.service
```

2. Add the following content to the file.
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \              # optional
--web.console.libraries=/etc/prometheus/console_libraries       # optional

[Install]
WantedBy=multi-user.target
```

3. Reload the systemd service.
```bash
systemctl daemon-reload
```

4. Start the prometheus service.
```bash
systemctl start prometheus
```

5. Check the prometheus service status (optional)
```bash
systemctl status prometheus
```

6. Enable the prometheus service so that it can get automatically started after boot
```bash
systemctl enable prometheus
```

## Step 5: Access Prometheus Web Interface
- Use the following URL to access Prometheus UI.
```bash
http://<prometheus-server-IP>:9090/graph     # replace prometheus-server-IP with the IP of you host (localhost) or VM
```
- You will see the following interface.

![prometheus-ui](images/prometheus-ui.png)
