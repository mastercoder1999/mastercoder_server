# Monitoring
## Infos
- [Promotheus](https://github.com/prometheus/prometheus) 
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Grafana](https://github.com/grafana/grafana)
## Promotheus
### Install
```
sudo useradd --no-create-home --shell /bin/false prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v3.10.0/prometheus-3.10.0.linux-amd64.tar.gz
tar xvf prometheus-3.10.0.linux-amd64.tar.gz
cd prometheus-3.10.0.linux-amd64/
```

### Préparation
```
sudo mv prometheus promtool /usr/local/bin/

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Service
```
sudo nano /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.listen-address=:9090

Restart=always

[Install]
WantedBy=multi-user.target
```
```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

sudo ufw allow 9090/tcp

```
http://192.168.1.169:9090/

## Node Exporter
### Install
```
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*.linux-amd64
sudo mv node_exporter /usr/local/bin/
```

### Service
```
sudo nano /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```
```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### Ajouter à la config Prometheus
```
sudo nano /etc/prometheus/prometheus.yml
```
Ajoutez ce job dans scrape_configs après le job Prometheus
```
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
        labels:
          app: "node_exporter"
          host: "goonerserver"
```
```
sudo systemctl restart prometheus
```
### Verif
```
sudo systemctl status prometheus
curl http://localhost:9100/metrics | head
```
Sur web aller sur http://192.168.1.169:9090/targets

Tu devrais voir:

- prometheus → UP
- node_exporter → UP

## Grafana (DASHBOARD)
### Install
```
sudo apt install -y apt-transport-https software-properties-common wget

sudo mkdir -p /etc/apt/keyrings

wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana -y
```

### Start
```
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### UFW
```
sudo ufw allow 3000/tcp
```
### Access
http://192.168.1.169:3000
admin / admin = default credentials. CHANGER LE MDP

### Config
- Setup le datasource Prometheus

- Installer le Dashboard (1860)

- Resultat

## Pihole-Exporter
Ajouter Pihole-Exporter au dashboard

### Install
```
cd /tmp

wget https://github.com/eko/pihole-exporter/releases/download/v1.2.0/pihole_exporter-linux-amd64

sudo mv pihole_exporter-linux-amd64 /usr/local/bin/pihole_exporter
sudo chmod +x /usr/local/bin/pihole_exporter
```
Test
```
pihole_exporter --help | head
```

### Service
```
sudo nano /etc/systemd/system/pihole-exporter.service
```
Remplacez le mot de passe pour le votre
```
[Unit]
Description=Pi-hole Exporter
After=network.target pihole-FTL.service

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/pihole_exporter \
  -pihole_hostname 127.0.0.1 \
  -pihole_password TON_MDP_PIHOLE \
  -web.listen-address :9617

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable pihole-exporter
sudo systemctl start pihole-exporter

systemctl status pihole-exporter --no-pager
curl http://localhost:9617/metrics | head
```
### Config
Ajouter la config pihole à Prometheus
```
sudo nano /etc/prometheus/prometheus.yml
```

```
  - job_name: "pihole"
    static_configs:
      - targets: ["localhost:9617"]
        labels:
          app: "pihole"
          host: "goonerserver"
```

```
sudo systemctl restart prometheus
```