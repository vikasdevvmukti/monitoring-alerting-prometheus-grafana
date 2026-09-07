# 📖 Setup Guide — Prometheus + Grafana Monitoring

### 🛠 Phase 1: Install Node Exporter (Metrics Collection)

```bash
cd ~
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64
./node_exporter &
```

Verify metrics are exposed:

```bash
curl localhost:9100/metrics
```

---

### 🛠 Phase 2: Install & Configure Prometheus

```bash
cd ~
wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
tar xvfz prometheus-2.54.1.linux-amd64.tar.gz
cd prometheus-2.54.1.linux-amd64
```

Edit `prometheus.yml` and add a scrape job for Node Exporter under `scrape_configs`:

```yaml
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

Start Prometheus:

```bash
./prometheus --config.file=prometheus.yml &
```

Verify at `http://your-ec2-ip:9090` → **Status → Targets**. The `node` job should show `UP`.

---

### 🛠 Phase 3: Install Grafana

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana -y
```

Start and enable the service:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

> If port 3000 is already used by another app, change Grafana's port in `/etc/grafana/grafana.ini`:
> ```ini
> [server]
> http_port = 3001
> ```
> Then restart: `sudo systemctl restart grafana-server`

Log in at `http://your-ec2-ip:3001` (default: `admin` / `admin`).

---

### 🛠 Phase 4: Connect Prometheus as a Data Source

1. Grafana → **Connections → Data sources → Add data source**
2. Select **Prometheus**
3. URL: `http://localhost:9090`
4. Click **Save & Test** — should confirm "Successfully queried the Prometheus API"

---

### 🛠 Phase 5: Import the Node Exporter Dashboard

1. Grafana → **Dashboards → New → Import**
2. Dashboard ID: `1860`
3. Select the Prometheus data source
4. Click **Import**

---

### 🛠 Phase 6: Create a CPU Alert Rule

1. Grafana → **Alerting → Alert rules → New alert rule**
2. Name: `High CPU Usage`
3. Query (PromQL):
   ```
   100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   ```
4. In the **Threshold** expression, set condition to **IS ABOVE** and value to `50`
5. Set **Pending period** to `2m`
6. Save the rule

---

### 🛠 Phase 7: Simulate an Incident

Install the load-testing tool:

```bash
sudo apt install stress -y
```

Generate CPU load for 3 minutes:

```bash
stress --cpu 4 --timeout 180
```

While this runs, watch:
- The **Grafana dashboard** — CPU graph should spike in real time
- **Alerting → Alert rules** — the `High CPU Usage` rule should move from `Normal` → `Alerting` after ~2 minutes

Once the `stress` command finishes, CPU usage drops and the alert automatically returns to `Normal` on the next evaluation cycle.
