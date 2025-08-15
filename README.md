# Export the provided Markdown content as a README.md file

markdown_content = """
# MySQL Cluster with Monitoring & Logging on ArvanCloud PaaS

## Introduction & Objective

This document outlines the complete workflow for deploying a **MySQL Master-Slave cluster** with integrated **logging** and **monitoring**, using **ArvanCloud PaaS**, **Helm**, and **Bitnami MySQL charts**. The implementation follows a **three-phase approach**:

1. **Phase 1 – MySQL Cluster Deployment**
    - Provision a Kubernetes-based MySQL primary-secondary cluster using Bitnami’s Helm chart.
    - Configure replication, security, performance tuning, and expose metrics for monitoring.
2. **Phase 2 – Monitoring Stack Setup**
    - Deploy a Prometheus & Grafana stack with **Ansible**, consuming MySQL metrics via `mysqld-exporter`.
    - Enable centralized performance tracking and visualization.
3. **Phase 3 – Logging Stack Setup (ELK)**
    - Implement an ELK stack (Elasticsearch, Logstash, Kibana) for centralized log collection and analysis.
    - Configure index management and optional ILM policies.

The goal is to build a **robust, observable, and maintainable** MySQL infrastructure that supports **real-time monitoring** and **centralized logging**, ensuring high availability, improved troubleshooting, and better performance insights.

---

# Phase 1 – Set-up MySQL Cluster on PaaS

<aside>
💡 **Project Description:**  
A MySQL cluster with Master-Slave topology with logging and monitoring on it.  
Requirements —> 2 CPU core, 4 GB RAM and 20GB SSD for each instance.  
We are using PaaS from ArvanCloud.
</aside>

## The Design

![MySQL Cluster Design](attachment:8a49f414-98a8-4c3d-8b59-70e9c8b5d8e2:telegram-cloud-photo-size-4-5823219000853776682-y.jpg)

### Major Tasks

- [ ] Set-up kubeconfig on local machine.
- [ ] Add Bitnami repo and pull MySQL Helm chart:  
  ```bash
  helm repo add bitnami https://charts.bitnami.com/bitnami

