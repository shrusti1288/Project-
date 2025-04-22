# Project 1-
Self-Healing Infrastructure with Prometheus, Alertmanager &amp; Ansible

Project Overview:

This project aims to create an automated self-healing infrastructure that detects failures in servers or services using Prometheus and Alertmanager, and automatically triggers remediation actions using Ansible. The main goal is to minimize manual intervention and downtime by automatically resolving known issues as soon as they are detected.


Key Technologies:

Prometheus – Monitoring and metrics collection

Alertmanager – Alert handling and routing

Ansible – Automation and configuration management

Node Exporter – System metrics exporter for Prometheus

Shell Scripts / Python – Optional custom actions



Architecture Diagram (Conceptual):

[Servers with Node Exporter] --> [Prometheus] --> [Alertmanager]
                                               |
                                        [Webhook Receiver]
                                               |
                                          [Ansible Playbook Trigger]
                                               |
                                        [Automated Remediation]



Main Components:

1. Prometheus

Collects metrics from the servers (CPU, memory, disk, service status).

Defines alerting rules (e.g., service down, high CPU usage).


2. Alertmanager

Receives alerts from Prometheus.

Groups, deduplicates, and routes alerts.

Sends alerts to a webhook receiver (which could be a Python script or other handler).


3. Webhook Receiver

Custom script or service (e.g., Flask API) that receives Alertmanager webhooks.

Parses alert details and triggers Ansible playbooks.


4. Ansible

Maintains a set of playbooks for common recovery actions:

Restart a failed service.

Reboot a server if unresponsive.

Clear disk space.


Triggered automatically by the webhook receiver.


Example Use Cases:

1. Service Down

Prometheus detects a failed service (e.g., Nginx).

Alertmanager sends an alert.

Webhook receiver triggers Ansible to restart the service.


2. High CPU Usage

Prometheus notices CPU > 90% for 5+ minutes.

Alert triggers an Ansible playbook to kill resource-heavy processes.


3. Low Disk Space

Prometheus detects disk space < 10%.

Ansible clears logs or temporary files.


Steps to Build:

1. Set up Prometheus to monitor system metrics with Node Exporter.


2. Define alert rules in Prometheus (alert.rules.yml).


3. Install and configure Alertmanager to handle those alerts.


4. Develop a webhook receiver to listen to alerts from Alertmanager.


5. Write Ansible playbooks for each recovery action.


6. Connect webhook receiver to execute relevant Ansible playbooks based on the alert content.


7. Test failure scenarios (e.g., kill a service, fill disk) to validate automation.


Optional Enhancements:

Add Slack/Email notifications via Alertmanager.

Integrate with Grafana for dashboards.

Log recovery actions to a centralized log (e.g., ELK stack).

Add role-based access or approval gates before triggering remediation (for critical actions).

Here’s a complete code setup for the Self-Healing Infrastructure with Prometheus, Alertmanager & Ansible project:


1. Prometheus Setup

a. Install Prometheus

wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-<version>.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*/

b. Basic prometheus.yml Configuration

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

rule_files:
  - "alert.rules.yml"


2. Node Exporter Setup (System Metrics)

a. Install Node Exporter

wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-<version>.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*/
./node_exporter

Make sure it’s running on port 9100.


3. Define Alert Rules – alert.rules.yml

groups:
  - name: system_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High CPU Usage on {{ $labels.instance }}"
          description: "CPU usage is above 90% for more than 2 minutes."
          
      - alert: NodeDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Node is down: {{ $labels.instance }}"


4. Alertmanager Setup

a. Basic alertmanager.yml Configuration

global:
  resolve_timeout: 1m

route:
  group_by: ['alertname']
  receiver: 'webhook'

receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://localhost:5000/alert'


5. Webhook Receiver (Python Flask Script)

webhook_receiver.py

from flask import Flask, request
import subprocess

app = Flask(__name__)

@app.route('/alert', methods=['POST'])
def alertmanager_webhook():
    alert = request.json['alerts'][0]
    alertname = alert['labels']['alertname']
    
    if alertname == "HighCPUUsage":
        subprocess.run(["ansible-playbook", "playbooks/fix_cpu.yml"])
    elif alertname == "NodeDown":
        subprocess.run(["ansible-playbook", "playbooks/reboot_node.yml"])
    
    return "Alert received", 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


6. Ansible Playbooks

a. playbooks/fix_cpu.yml

- name: Fix high CPU usage
  hosts: all
  become: true
  tasks:
    - name: Kill high CPU usage processes (example: stress)
      shell: "pkill -f stress"
      ignore_errors: yes

b. playbooks/reboot_node.yml

- name: Reboot node
  hosts: all
  become: true
  tasks:
    - name: Reboot the server
      reboot:
        reboot_timeout: 300


7. Run Everything

Start Prometheus:

./prometheus --config.file=prometheus.yml

Start Node Exporter:

./node_exporter

Start Alertmanager:

./alertmanager --config.file=alertmanager.yml

Start Webhook Flask App:

python3 webhook_receiver.py

Ensure Ansible is installed and /etc/ansible/hosts contains the target node(s).

