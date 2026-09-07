# 📊 Server Monitoring & Alerting with Prometheus + Grafana

**Objective:** Set up a full observability stack on an AWS EC2 instance — collecting system metrics, visualizing them on live dashboards, and configuring threshold-based alerting to detect performance issues before they become outages.

---

### 🚀 Implementation Strategy

1. **Metrics Collection:** Deployed Node Exporter to expose system-level metrics (CPU, memory, disk, network) from the EC2 instance.
2. **Metrics Storage & Scraping:** Configured Prometheus to scrape and store these metrics on a regular interval.
3. **Visualization:** Connected Grafana to Prometheus as a data source and imported a pre-built dashboard for a complete system overview.
4. **Alerting:** Created a Grafana-managed alert rule to detect sustained high CPU usage and trigger a state change in real time.
5. **Incident Simulation:** Used a synthetic CPU load test to validate the entire pipeline — from metric collection to alert firing to recovery.

---

### 📊 Visual Evidence & Documentation

#### 1. Node Exporter — Metrics Exposed
Raw system metrics available at the `/metrics` endpoint before Prometheus even scrapes them.
<img width="1920" height="1007" alt="Screenshot from 2026-09-07 16-56-53" src="https://github.com/user-attachments/assets/ed5d5681-0d21-4888-ab93-f8cc080e0ddc" />


#### 2. Prometheus — Target Health Check
Prometheus successfully scraping the Node Exporter target, confirmed via the Targets page (`UP` status).
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 17-04-30" src="https://github.com/user-attachments/assets/1de24528-1d1a-4be7-a579-32a798ab0255" />


#### 3. Grafana — Data Source Connected
Prometheus successfully added as a Grafana data source.
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 17-52-02" src="https://github.com/user-attachments/assets/b88947e2-ee1d-4f64-80e5-2330f5cf2a39" />


#### 4. Grafana — Live System Dashboard
Imported the community Node Exporter dashboard (ID: 1860), showing real-time CPU, memory, network, and disk usage. The CPU panel below captures a load spike from the incident simulation (Basic CPU graph, red area).
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 17-53-53" src="https://github.com/user-attachments/assets/64322bc1-c9e2-4603-aa87-b176e83078a4" />


#### 5. Alert Triggered — "Alerting" State
A custom alert rule (`High CPU Usage`, threshold: CPU > 50%, pending period: 2 minutes) transitions to the **Alerting** state once sustained load is detected.
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 18-17-57" src="https://github.com/user-attachments/assets/f5aa7926-38f4-48a2-b4c8-926ba7670c00" />


#### 6. Alert Instance Detail
The alert instance view showing the exact label set (`instance="localhost:9100"`) and timestamp of when the alert began firing.
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 18-17-50" src="https://github.com/user-attachments/assets/7997cc96-f3c7-4f5e-b8b2-495ca50ccaa2" />


#### 7. Alert Recovered — Back to "Normal"
Once the simulated load subsided, the alert automatically transitioned back to **Normal**, with the graph confirming CPU usage dropping below threshold.
<img width="1914" height="1011" alt="Screenshot from 2026-09-07 18-20-54" src="https://github.com/user-attachments/assets/41c372be-516c-43c4-9c1b-fbea923865a7" />


---

### 🧾 Incident Report (Simulated)

| Field | Detail |
|---|---|
| **Trigger** | Synthetic CPU load generated using the `stress` tool (`stress --cpu 4 --timeout 180`) |
| **Detection** | Grafana alert rule evaluated every 2 minutes; entered "Alerting" state after CPU sustained above the 50% threshold for the configured pending period |
| **Confirmation** | Live dashboard graph showed a clear CPU spike correlating with the alert timestamp |
| **Resolution** | Load ended automatically after the timeout; CPU usage dropped, and the alert returned to "Normal" on the next evaluation cycle |
| **Detection Time** | ~2 minutes from threshold breach to alert state change |

This exercise validates that the monitoring pipeline correctly detects, reports, and clears real performance events — the same mechanism that would apply to genuine production incidents (e.g., traffic spikes, runaway processes, resource leaks).

---

### 🧰 Tech Stack & Tools
* **Metrics Exporter:** Prometheus Node Exporter
* **Metrics Storage & Scraping:** Prometheus
* **Visualization & Alerting:** Grafana
* **Infrastructure:** AWS EC2 (Ubuntu)
* **Load Testing:** `stress`

---

### 📁 Project Structure
* **/configs** — Prometheus scrape configuration (`prometheus.yml`).
* **/docs** — [Step-by-Step Setup Guide](docs/setup-guide.md) covering installation, configuration, and alert rule creation.
