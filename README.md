# monitoring-assesment

UI: http://localhost:9090

Grafana UI: http://localhost:3000 (user/pass: admin/admin)

/
├── ansible/
│ ├── install_node_exporter.yml # Ansible playbook to install Node Exporter
│ └── render_prometheus_config.yml # Ansible playbook to render Prometheus config
├── prometheus/
│ ├── prometheus.yml.j2 # Prometheus config template with dynamic targets
│ ├── alertmanager.yml # Alertmanager configuration file for email alerts
│ └── alerts.yml # Alerting rules for Prometheus
├── docker-compose.yml # Docker Compose file to bring up Prometheus, Grafana, and Alertmanager
└── README.md # Documentation for the setup

The solution uses Prometheus for metric collection, Node Exporter for gathering host-level metrics, and Grafana for visualization. Alertmanager is used to send email notifications when critical alerts are triggered.

This setup is designed with scalability in mind. As the number of servers increases, we only need to update the Prometheus configuration as targets are dynamically generated using Ansible templating ({% for host in groups['node_exporters'] %}), meaning this file is automatically updated to reflect the hosts defined in your Ansible inventory. This way, Prometheus will automatically know the target nodes without having to manually update this file.
we can also modify the alert file according to our needs



**Pre-requisites**
Docker and Docker Compose installed on the machine where Prometheus and Grafana will run.
Ansible installed on the machine from which you’ll configure the servers.
Access to the target servers where Node Exporter will be installed


**Design Approach Overview**
The solution is designed to:
Collect critical metrics from multiple servers using Node Exporter. 
Use Prometheus to scrape these metrics and store them in a time-series database. Prometheus.yml using the promethus.yml.j2 template.
Visualize the data with Grafana dashboards. 
Set up alerting to notify you of critical conditions like high CPU usage, memory usage, disk space usage, etc1.The alerting section tells Prometheus to send alerts to Alertmanager at alertmanager:9093 
Configure Ansible Inventory to Install Node Exporter on All Servers
Configure Prometheus, Modify the prometheus.yml.j2 template to include the IP addresses of all your servers

Steps

# Step 1 :  Add target servers under node_exporter group in hosts.ini

# Step 2: Install Node Exporter on all target servers
ansible-playbook -i hosts.ini install_node_exporter.yml

# Step 3: Generate prometheus config file" prometheus.yml using jinja template
ansible-playbook -i hosts.ini render_prometheus_config.yml

# Step 4: Run docker-compose file to start Prometheus, Alertmanager and Grafana
cd prometheus/
docker-compose up -d
