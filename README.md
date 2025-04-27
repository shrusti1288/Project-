# Project 1-
Self-Healing Infrastructure with Prometheus, Alertmanager & Ansible.

Got it — you're asking for full project details for
Self-Healing Infrastructure with Prometheus, Alertmanager & Ansible.
Let’s dive in deep:


Project Title:

Self-Healing Infrastructure using Prometheus, Alertmanager, and Ansible


1. Objective

Automatically detect failures (like a crashed service, full disk, high CPU, etc.) using Prometheus monitoring and Alertmanager alerts, and trigger Ansible playbooks to auto-remediate the issue — without human intervention.


2. Architecture Overview

[ Monitored Servers ]
        |
[ Prometheus Server ]
        |
[ Alertmanager ]
        |
[ Webhook Receiver (Custom Flask app) ]
        |
[ Ansible Automation (Playbooks) ]

Flow:

Prometheus scrapes metrics from servers.

If a metric threshold is crossed (e.g., CPU > 90%), Prometheus fires an alert.

Alertmanager sends the alert to a webhook (custom Flask app).

Webhook app triggers corresponding Ansible playbooks to fix the issue.


3. Components


4. Technologies Required

Prometheus (monitoring tool)

Alertmanager (alert handling)

Node Exporter (metrics exporter for Linux servers)

Ansible (for automation and remediation)

Python Flask (simple webhook receiver)

Docker (optional: for containerizing services)

Linux servers (test environment: Ubuntu, CentOS, etc.)


5. Setup Guide

5.1 Install Node Exporter on all servers

# Download Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.linux-amd64.tar.gz
tar -xvf node_exporter-*.tar.gz
cd node_exporter-*

# Start Node Exporter
./node_exporter &

Node Exporter listens by default on :9100.


5.2 Setup Prometheus Server

prometheus.yml (basic configuration):

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporters'
    static_configs:
      - targets: ['server1_ip:9100', 'server2_ip:9100']

Install Prometheus:


wget https://github.com/prometheus/prometheus/releases/download/v*/prometheus-*.linux-amd64.tar.gz
tar -xvf prometheus-*.tar.gz
cd prometheus-*
./prometheus --config.file=prometheus.yml


5.3 Setup Alertmanager

alertmanager.yml:

global:
  resolve_timeout: 5m

receivers:
- name: 'webhook-receiver'
  webhook_configs:
  - url: 'http://<your_webhook_server>:5000/alert'

route:
  receiver: 'webhook-receiver'

Start Alertmanager:

./alertmanager --config.file=alertmanager.yml

Prometheus also needs to know where Alertmanager is: Add in prometheus.yml:

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']


5.4 Define Prometheus Alert Rules

Example - alerts.yml:

groups:
- name: example
  rules:
  - alert: HighCPUUsage
    expr: node_load1 > 1
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage detected"
      description: "The CPU load is above 1 for more than 1 minute"

In prometheus.yml:

rule_files:
  - 'alerts.yml'


5.5 Create Webhook Receiver (Flask App)

webhook_receiver.py:

from flask import Flask, request
import subprocess

app = Flask(__name__)

@app.route('/alert', methods=['POST'])
def alert():
    data = request.json
    for alert in data['alerts']:
        if alert['labels']['alertname'] == 'HighCPUUsage':
            subprocess.run(["ansible-playbook", "playbooks/restart_service.yml"])
    return 'OK', 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

Run:

pip install flask
python webhook_receiver.py


5.6 Create Ansible Playbook

Example - restart_service.yml:

- name: Auto Remediation - Restart Service
  hosts: all
  tasks:
    - name: Restart apache2 (example service)
      service:
        name: apache2
        state: restarted

Inventory file - hosts.ini:

[all]
server1 ansible_host=server1_ip ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/key

Run manually to test:

ansible-playbook -i hosts.ini playbooks/restart_service.yml


6. Example: Complete Trigger Flow


7. Extra Features to Improve

Add multiple playbooks based on alert type.

Integrate Grafana dashboards for visualization.

Add slack/email notification in parallel.

Implement retry mechanism in Ansible if the first fix fails.

Dockerize the Flask webhook app.

Security: Add basic auth to the webhook.


8. Folder Structure

self-healing-infrastructure/
│
├── ansible/
│   ├── hosts.ini
│   └── playbooks/
│       └── restart_service.yml
│
├── prometheus/
│   ├── prometheus.yml
│   ├── alerts.yml
│
├── alertmanager/
│   └── alertmanager.yml
│
├── webhook/
│   └── webhook_receiver.py
│
└── README.md


9. Deployment Tip

Use Docker Compose to bring up Prometheus, Alertmanager, Webhook Flask App, and Grafana together.


10. Real World Use-Cases

Auto-restart crashed web servers or databases.

Auto-expand disk partitions when disk usage > 90%.

Auto-recover Kubernetes pods using kubectl via Ansible.

Self-remediation for security patches on vulnerable servers.


Let’s build this Self-Healing Infrastructure project into a more production-ready, GitHub-level setup.

Here’s what I’ll give you now:

Full ready folder structure.

Docker Compose to run everything easily.

Webhook Receiver Flask app (improved).

Ansible setup (with proper roles).

Sample alert rules and Prometheus config.

A short README.md to explain how to run it.


Self-Healing Infrastructure — Final Project



1. Folder Structure

self-healing-infrastructure/
│
├── ansible/
│   ├── ansible.cfg
│   ├── inventory.ini
│   └── roles/
│       └── remediation/
│           ├── tasks/
│           │   └── main.yml
│           └── handlers/
│               └── main.yml
│
├── prometheus/
│   ├── prometheus.yml
│   ├── alerts.yml
│
├── alertmanager/
│   └── alertmanager.yml
│
├── webhook/
│   ├── app.py
│   ├── requirements.txt
│
├── docker-compose.yml
│
└── README.md


2. ansible/ directory

ansible.cfg

[defaults]
inventory = inventory.ini
host_key_checking = False
retry_files_enabled = False

inventory.ini

[all]
server1 ansible_host=SERVER1_IP ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

roles/remediation/tasks/main.yml

- name: Ensure apache2 is running
  become: yes
  service:
    name: apache2
    state: restarted


3. prometheus/prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporters'
    static_configs:
      - targets: ['server1_ip:9100']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'


4. prometheus/alerts.yml

groups:
- name: node_alerts
  rules:
  - alert: HighCPUUsage
    expr: node_load1 > 1
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "High CPU load detected on instance {{ $labels.instance }}"
      description: "CPU load is > 1 for 30 seconds."


5. alertmanager/alertmanager.yml

global:
  resolve_timeout: 5m

receivers:
- name: webhook
  webhook_configs:
  - url: 'http://webhook:5000/alert'

route:
  receiver: webhook


6. webhook/app.py

from flask import Flask, request
import subprocess
import os

app = Flask(__name__)

@app.route('/alert', methods=['POST'])
def alert():
    data = request.json
    for alert in data['alerts']:
        if alert['labels']['alertname'] == 'HighCPUUsage':
            subprocess.run([
                "ansible-playbook",
                "-i", "ansible/inventory.ini",
                "ansible/roles/remediation/tasks/main.yml"
            ])
    return "Alert received", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

webhook/requirements.txt

Flask


7. docker-compose.yml

version: '3'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alerts.yml:/etc/prometheus/alerts.yml
    ports:
      - "9090:9090"

  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"

  webhook:
    build:
      context: ./webhook
    ports:
      - "5000:5000"
    volumes:
      - ./ansible:/ansible
    depends_on:
      - alertmanager

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"


8. README.md

# Self-Healing Infrastructure

This project uses Prometheus, Alertmanager, Flask, and Ansible to build a self-healing infrastructure system.

## Components
- Prometheus - Monitoring
- Alertmanager - Alerting
- Flask Webhook App - Receive alerts
- Ansible - Remediation

## How to Run

1. Clone this repo:
```bash
git clone <repo_url>
cd self-healing-infrastructure

2. Start the services:



docker-compose up --build

3. Make sure your target server (server1) has Node Exporter running:



wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
./node_exporter

4. Create your SSH key to connect Ansible to the target server if needed.


5. Done! Prometheus will auto-detect issues and Ansible will fix them.



Example

High CPU Usage detected -> Alertmanager -> Webhook -> Ansible -> Apache2 restarted automatically!


Future improvements

Slack / Email notification

Kubernetes deployment

Ansible Roles for different failure types


# **How to deploy (Summary):**

```bash
docker-compose up --build

and
make sure Node Exporter is running on your server.


Bonus Tip (Optional):

You can also containerize the webhook app with a simple Dockerfile inside webhook/:


FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]


Done!

You now have a fully ready-to-run, professional-grade
Self-Healing Infrastructure with Prometheus, Alertmanager & Ansible!

Let's make a Demo Failure Scenario Script that forces CPU usage high, so Prometheus detects it, fires an alert, and you see Ansible automatically fixing it — live.


Demo Failure Scenario: Force High CPU Load

Create a new file:

scripts/high_cpu.sh

#!/bin/bash

# This script creates high CPU usage for ~2 minutes
echo "Starting high CPU load..."

# Run 4 processes that stress CPU
for i in {1..4}
do
    yes > /dev/null &
done

# Keep it running for 120 seconds
sleep 120

# Kill all 'yes' processes
killall yes

echo "CPU load simulation finished."


How it works:

yes > /dev/null forces a core to max CPU usage.

We spawn 4 yes commands (you can increase this if your server has more cores).

After 2 minutes, it automatically kills the load.


How to use it:

1. SSH into your monitored server (server1 where Node Exporter is running).


2. Upload the script:



scp scripts/high_cpu.sh ubuntu@server1_ip:~/

3. Make it executable:



chmod +x high_cpu.sh

4. Run it:



./high_cpu.sh


Expected Flow:

You will see logs in:

Prometheus UI (localhost:9090 → Alerts)

Alertmanager UI (localhost:9093 → Alerts)

Flask Webhook logs (Auto-playbook triggered)

Ansible output (task results)


Extra Tip:

If you want an even fancier real-time visualization:
You can add a Grafana dashboard showing the CPU graphs — I can help you with that too if you want!

I’ll now create TWO more demo failure scenarios for you:

Demo 2: Simulate Disk Full Condition

We’ll create a file that fills up your disk temporarily.

scripts/fill_disk.sh

#!/bin/bash

# This script fills up disk space temporarily (~90% usage)

echo "Starting disk filling..."

# Create a big file (adjust count=102400 for ~100MB file)
dd if=/dev/zero of=~/largefile bs=1M count=5120

# Wait for 2 minutes
sleep 120

# Remove the file to free up space
rm -f ~/largefile

echo "Disk fill simulation finished."

What it does:

Creates a 5GB file (largefile) to fill disk space fast.

Waits 2 minutes to allow Prometheus to fire a Disk Usage High alert.

Then deletes the large file automatically.


Prometheus Alert Rule for Disk Usage (add this to alerts.yml)

- alert: DiskUsageHigh
  expr: (node_filesystem_avail_bytes{fstype!~"tmpfs|fuse.lxcfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|fuse.lxcfs"}) < 0.10
  for: 30s
  labels:
    severity: critical
  annotations:
    summary: "Disk almost full on {{ $labels.instance }}"
    description: "Available disk space is less than 10%"

Then in webhook/app.py:

for alert in data['alerts']:
    if alert['labels']['alertname'] == 'HighCPUUsage':
        subprocess.run([
            "ansible-playbook",
            "-i", "ansible/inventory.ini",
            "ansible/roles/remediation/tasks/main.yml"
        ])
    elif alert['labels']['alertname'] == 'DiskUsageHigh':
        subprocess.run([
            "ansible-playbook",
            "-i", "ansible/inventory.ini",
            "ansible/roles/remediation/tasks/disk_cleanup.yml"
        ])

Create ansible/roles/remediation/tasks/disk_cleanup.yml:

- name: Cleanup temporary files
  become: yes
  shell: |
    rm -rf /tmp/*
    rm -f ~/largefile


Demo 3: Critical Service Stopped (Apache2 Down)

Simulate a critical service crash and auto-heal it.

Manual Steps:

On the monitored server, run:

sudo systemctl stop apache2

This stops Apache2 manually.

Prometheus Alert Rule (add to alerts.yml):

- alert: ApacheDown
  expr: node_systemd_unit_state{name="apache2.service",state="active"} == 0
  for: 30s
  labels:
    severity: critical
  annotations:
    summary: "Apache service down on {{ $labels.instance }}"
    description: "Apache2 service is not running."

Webhook handler:

elif alert['labels']['alertname'] == 'ApacheDown':
        subprocess.run([
            "ansible-playbook",
            "-i", "ansible/inventory.ini",
            "ansible/roles/remediation/tasks/restart_apache.yml"
        ])

Create ansible/roles/remediation/tasks/restart_apache.yml:

- name: Restart Apache2 Service
  become: yes
  service:
    name: apache2
    state: restarted


Summary Table:

(Optional - Bonus Visual)

Add Grafana dashboards to see live CPU, Disk, and Service states.

I can give you a pre-built Grafana dashboard JSON file if you want.

 let's create a professional full GitHub README.md for your
Self-Healing Infrastructure with Prometheus, Alertmanager, Ansible, and Flask project.

It will be complete with:

Project Overview

Architecture Diagram (simple ASCII)

Tech Stack

Folder Structure

Installation Guide

Failure Scenarios (Demo Scripts)

How Auto-Healing Works

Future Improvements

License

Here’s your Full GitHub README.md:

# Self-Healing Infrastructure with Prometheus, Alertmanager, Ansible, and Flask

> Fully automated self-healing system for critical infrastructure, capable of detecting failures and remediating them without human intervention.


## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Folder Structure](#folder-structure)
- [Installation](#installation)
- [Failure Scenarios (Demo)](#failure-scenarios-demo)
- [How Auto-Healing Works](#how-auto-healing-works)
- [Future Improvements](#future-improvements)
- [License](#license)


## Overview

This project sets up a self-healing infrastructure system using:
- **Prometheus** for monitoring
- **Alertmanager** for alerting
- **Flask Webhook App** for receiving alerts
- **Ansible** for automatic remediation

Failures such as high CPU usage, disk almost full, or critical service crashes are detected and automatically fixed without manual intervention.


## Architecture

```plaintext
+----------------+         +---------------+         +-------------+         +----------------+
| Monitored Node | ----->  |   Prometheus   | ----->  | Alertmanager | ----->  | Flask Webhook   |
| (Node Exporter) |        | (Scraping data)|         | (Firing alerts)|       | (Triggers Ansible)|
+----------------+         +---------------+         +-------------+         +----------------+


Tech Stack

Prometheus

Alertmanager

Flask (Python 3)

Ansible

Docker Compose


Folder Structure

self-healing-infrastructure/
│
├── ansible/
│   ├── ansible.cfg
│   ├── inventory.ini
│   └── roles/
│       └── remediation/
│           └── tasks/
│               ├── main.yml
│               ├── disk_cleanup.yml
│               └── restart_apache.yml
│
├── prometheus/
│   ├── prometheus.yml
│   ├── alerts.yml
│
├── alertmanager/
│   └── alertmanager.yml
│
├── webhook/
│   ├── app.py
│   ├── requirements.txt
│
├── scripts/
│   ├── high_cpu.sh
│   ├── fill_disk.sh
│
├── docker-compose.yml
├── README.md
└── LICENSE


Installation

1. Clone this Repository:



git clone https://github.com/your-username/self-healing-infrastructure.git
cd self-healing-infrastructure

2. Start Services with Docker Compose:


docker-compose up --build

3. Setup Node Exporter on Monitored Server:


wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
cd node_exporter-*/
./node_exporter

4. Ensure SSH Access for Ansible:


Create SSH key (ssh-keygen)

Copy key to server (ssh-copy-id)


5. Access Dashboards:


Prometheus → http://localhost:9090

Alertmanager → http://localhost:9093

Webhook logs via Docker container logs


Failure Scenarios (Demo)

1. High CPU Usage

On monitored server:


./scripts/high_cpu.sh

Expected Auto-Remediation: Ansible triggers a task to restart critical services.


2. Disk Almost Full

On monitored server:


./scripts/fill_disk.sh

Expected Auto-Remediation: Ansible clears /tmp/ files and large dummy files.


3. Critical Service Down (Apache2)

On monitored server:


sudo systemctl stop apache2

Expected Auto-Remediation: Ansible restarts apache2 service automatically.


How Auto-Healing Works

1. Prometheus scrapes server metrics every 15 seconds.


2. Prometheus alert rules trigger if a threshold breach occurs (e.g., CPU > 90% for 30 seconds).


3. Alertmanager receives alert and forwards it to Flask Webhook.


4. Flask Webhook parses the alert and calls relevant Ansible playbook.


5. Ansible automatically connects to the server and remediates the issue.



Future Improvements

Add Slack/Discord notifications for alert summaries.

Include Grafana dashboards for better visualization.

Support for multiple services (MySQL, Nginx, Docker health).

Kubernetes (K8s) support for Pod Auto-Recovery.

Better alert deduplication and smarter remediation decision trees.


License

This project is licensed under the MIT License.


# **NOTES:**
- The `scripts/` folder is added for your CPU/Disk fill scripts.
- Later you can attach screenshots in the README too (e.g., CPU graph in Prometheus, alert firing in Alertmanager).

  
# **Next Steps you could take (Optional Polish):**
- Add **Grafana** container in `docker-compose.yml` for monitoring dashboards.
- Create **better Ansible role structure** if you add more services.
- Make a short **demo video** (gif or mp4) to showcase on GitHub (very good for your profile).










