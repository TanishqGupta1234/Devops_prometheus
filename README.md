# **Monitoring with Prometheus and Grafana**


## **1. Install Node Exporter on Linux/WSL/Docker**
Node Exporter collects system metrics and exposes them for Prometheus.

### **Steps:**
1. **Download and extract Node Exporter**:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
```
![Screenshot 2025-04-17 105728](https://github.com/user-attachments/assets/be20f4a2-6a2b-42fb-b3e3-04a6736bea34)

2. **Run Node Exporter**:
```bash
./node_exporter
```
- This starts Node Exporter on `http://localhost:9100`.
![Screenshot 2025-04-17 105837](https://github.com/user-attachments/assets/140fe425-e273-4c3d-92ab-ee35bede6959)


3. **Verify Metrics**:
- Open in browser: [http://localhost:9100/metrics](http://localhost:9100/metrics)
- Or check via `curl`:
    ```bash
    curl http://localhost:9100/metrics
    curl http://localhost:9100/metrics | grep "node_"
    ```
![Screenshot 2025-04-17 105854](https://github.com/user-attachments/assets/8c409c54-f435-4885-a8ea-0c029e9b24ee)


4. **Note the WSL IP (if needed)**:
- If Prometheus runs outside WSL (e.g., Windows host or Docker), get WSL's IP:
    ```bash
    hostname -I
    ```
- Replace `localhost` with this IP in `prometheus.yml` later.

---

## **2. Start Prometheus & Grafana with Docker Compose**
Prometheus scrapes metrics, and Grafana visualizes them.

### **Steps:**
1. **Create `docker-compose.yml`**:
```yaml
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=grafana
    volumes:
      - ./grafana:/etc/grafana/provisioning/datasources
volumes:
  prom_data:
```

2. **Configure `prometheus.yml`**:
```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s

scrape_configs:
- job_name: prometheus
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost:9090
  
- job_name: node_exporter
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    # - localhost:9100 # you migth want to change this to your host IP, if running in node exporter is running in wsl use `hostname -I` to get IP
    - 172.27.200.56:9100 # you migth want to change this to your host IP, if running in node exporter is running in wsl use `hostname -I` to get IP

```

3. **Start Services**:
```bash
docker-compose up -d
```
![Screenshot 2025-04-17 111039](https://github.com/user-attachments/assets/c21d2653-2073-4237-8f36-8a89a4c7dcb7)


4. **Verify Prometheus**:
- Open [http://localhost:9090](http://localhost:9090)
- Check targets: **Status → Targets** → Ensure `node-exporter` is **UP**.

---
![Screenshot 2025-04-17 112222](https://github.com/user-attachments/assets/137fc8db-00dc-4fcb-9722-52a503372b10)

![Screenshot 2025-04-17 124853](https://github.com/user-attachments/assets/bdf2d3ce-cdbb-4c71-ab68-c5125ddb29e7)


## **3. Set Up Grafana Dashboard**
Grafana provides visualization for collected metrics.

### **Steps:**
1. **Access Grafana**:
- Open [http://localhost:3000](http://localhost:3000)
- Default login: `admin` / `admin`
![Screenshot 2025-04-17 124952](https://github.com/user-attachments/assets/d0b524cc-89c3-4398-aedf-e40daab99a77)
![Screenshot 2025-04-17 112330](https://github.com/user-attachments/assets/5075f106-9642-4d9f-b5ca-c043348c07f2)


2. **Add Prometheus Data Source**:
- **Configuration → Data Sources → Add Prometheus**
- URL: `http://prometheus:9090` (or `http://<Prometheus_IP>:9090` if not in Docker)
![Screenshot 2025-04-17 112557](https://github.com/user-attachments/assets/57c67fef-49ba-4f8e-a887-0752d3bf2eb3)
![Screenshot 2025-04-17 112753](https://github.com/user-attachments/assets/1b39f8b5-ab25-497e-87f7-6ff3ee538f24)


3. **Import Node Exporter Dashboard**:
- **Create (+) → Import → Dashboard ID `1860`**  
    (or download [Node Exporter Full Dashboard](https://grafana.com/grafana/dashboards/1860))
- Select Prometheus data source → **Import**.
![Screenshot 2025-04-17 112841](https://github.com/user-attachments/assets/959c22d5-0fff-418c-b779-98c174adbfce)


4. **View Metrics**:
- The dashboard will now display CPU, memory, disk, and network metrics.

---
![WhatsApp Image 2025-04-17 at 21 35 02](https://github.com/user-attachments/assets/5cab920c-1fdb-4e9e-9529-56b1662c9bc8)


## **Troubleshooting**
- **Connection Issues?**
- Ensure `node_exporter` is running (`curl http://localhost:9100/metrics`).
- Check `prometheus.yml` uses the correct IP (`hostname -I` in WSL).
- Disable firewalls blocking port `9100` (Windows Defender/WSL).

- **Prometheus Not Scraping?**
- Restart services: `docker-compose restart`.
- Check logs: `docker-compose logs prometheus`.

---

## **Summary**
**Node Exporter** installed and running (`:9100/metrics`).  
**Prometheus** scraping metrics (`:9090/targets`).  
**Grafana** dashboard imported (`:3000`).  

Now you have a full monitoring stack! 



---

#RAW Instructions

# 1 Install prometheus exporter on test linux system/wsl/docker container

read instructions [https://prometheus.io/docs/guides/node-exporter/](https://prometheus.io/docs/guides/node-exporter/)

````bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
./node_exporter

echo "visit http://localhost:9100/metrics"
```



after this tutorial check metrics at http://localhost:9100/metrics

or execute

```bash
curl http://localhost:9100/metrics
curl http://localhost:9100/metrics | grep "node_"
```

It will show metrics however, you might need to use actual IP of node exporter, use `hostname -I` to get IP of WSL, and replace `- targets: <actual ip of WSL/node_exporter>` in prometheus.yml

# 2 Start prometheus and grafana using docker compose
now execute `docker-compose up -d`

check prometheus data collection database server
[localhost:9090](localhost:9090)

go to Prometheus >> status >> targets to view status of targets. If status is UP, you can continue with grafana.


View grafana dashboard at 
[localhost:3000](localhost:3000)

follow 
+ >> New Dashboard,

you can create your own dashboard, however, it is easier to import from existing templates,
>> import dashboard >> Discard

visit grafana.com/dashboards for available templates,
but for node exporter use template at [1860-node-exporter-full/](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)

>> download JSON, copy it and add it on previous pannel `JSON model`.
>> Load,

> Observe shown metrics.
