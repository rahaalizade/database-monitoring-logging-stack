# MySQL Master-Slave Cluster with Monitoring & Logging

A comprehensive deployment guide for setting up a highly available MySQL cluster with integrated monitoring and logging capabilities using ArvanCloud PaaS, Kubernetes, and modern observability tools.

## 🏗️ Architecture Overview

![MySQL Cluster Architecture](docs/architecture-diagram.jpg)

This project implements a production-ready MySQL infrastructure with:

- **MySQL Master-Slave Cluster**: High availability database setup
- **Prometheus & Grafana**: Real-time monitoring and visualization
- **ELK Stack**: Centralized logging and log analysis
- **ArvanCloud PaaS**: Kubernetes-based container orchestration

## 📋 Table of Contents

- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Phase 1: MySQL Cluster Deployment](#phase-1-mysql-cluster-deployment)
- [Phase 2: Monitoring Stack Setup](#phase-2-monitoring-stack-setup)
- [Phase 3: Logging Stack Setup](#phase-3-logging-stack-setup)
- [Configuration](#-configuration)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

- **High Availability**: Master-slave MySQL replication for data redundancy
- **Performance Monitoring**: Comprehensive metrics collection via mysqld-exporter
- **Real-time Dashboards**: Pre-configured Grafana dashboards for MySQL insights
- **Centralized Logging**: ELK stack for log aggregation and analysis
- **Security**: Proper secret management and network isolation
- **Scalability**: Kubernetes-native deployment for easy scaling
- **Automation**: Ansible playbooks for infrastructure as code

## 📦 Prerequisites

### System Requirements

**Per MySQL Instance:**
- 2 CPU cores
- 4 GB RAM
- 20 GB SSD storage

**Monitoring Server:**
- 2 CPU cores
- 4 GB RAM
- 50 GB storage

**Logging Server (ELK):**
- 4 CPU cores
- 8 GB RAM
- 100 GB storage

### Tools & Dependencies

- **Kubernetes Cluster** (ArvanCloud PaaS)
- **Helm** (v3.x)
- **kubectl** configured with cluster access
- **Ansible** (v2.9+)
- **Docker** (for local development)

## 🏛️ Architecture

The project follows a three-tier architecture:

1. **Data Layer**: MySQL Master-Slave cluster with automated failover
2. **Monitoring Layer**: Prometheus for metrics collection, Grafana for visualization
3. **Logging Layer**: Elasticsearch for storage, Logstash for processing, Kibana for analysis

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   MySQL Master  │────│  MySQL Slave    │    │   Load Balancer │
│   (Primary)     │    │  (Secondary)    │────│   (Public IP)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Prometheus    │────│    Grafana      │    │   ELK Stack     │
│   (Metrics)     │    │  (Dashboard)    │    │   (Logging)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/mysql-cluster-monitoring.git
   cd mysql-cluster-monitoring
   ```

2. **Configure kubectl**
   ```bash
   # Download and configure your kubeconfig from ArvanCloud
   export KUBECONFIG=/path/to/your/kubeconfig
   kubectl get nodes
   ```

3. **Add Bitnami Helm repository**
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```

4. **Deploy the complete stack**
   ```bash
   # Phase 1: MySQL Cluster
   ./scripts/deploy-mysql.sh
   
   # Phase 2: Monitoring
   ./scripts/deploy-monitoring.sh
   
   # Phase 3: Logging
   ./scripts/deploy-logging.sh
   ```

## Phase 1: MySQL Cluster Deployment

### 🎯 Objectives

Deploy a Kubernetes-based MySQL primary-secondary cluster using Bitnami's Helm chart with:
- Automated replication setup
- Performance tuning
- Security configurations
- Metrics exposure

### 📝 Tasks Checklist

- [ ] Set up kubeconfig on local machine
- [ ] Add Bitnami Helm repository
- [ ] Create MySQL Helm chart configuration
- [ ] Create `mysql-credentials` secret
- [ ] Install Helm chart with custom values
- [ ] Configure mysqld-exporter for metrics
- [ ] Verify container health and database connectivity
- [ ] Set up load balancer for public access

### 🔧 Deployment Steps

1. **Create the MySQL credentials secret**
   ```bash
   kubectl create secret generic mysql-credentials \
     --from-literal=mysql-root-password='your-root-password' \
     --from-literal=mysql-password='your-user-password' \
     --from-literal=mysql-replication-password='your-replication-password'
   ```

2. **Deploy MySQL cluster**
   ```bash
   helm install mysql bitnami/mysql -f values.yaml
   ```

3. **Verify deployment**
   ```bash
   kubectl get pods -l app.kubernetes.io/name=mysql
   kubectl logs mysql-primary-0
   ```

### 🔗 Load Balancer Configuration

Configure public IP allocation in ArvanCloud dashboard:
- **Ports**: 3306 (MySQL), 9104 (Metrics)
- **Protocol**: TCP
- **Type**: Load Balancer

## Phase 2: Monitoring Stack Setup

### 🎯 Objectives

Deploy Prometheus & Grafana stack using Ansible to monitor MySQL performance and cluster health.

### 📁 Project Structure

```
monitoring/
├── inventories/
│   └── hosts.yml
├── playbooks/
│   └── monitoring.yml
└── roles/
    ├── observer/
    │   ├── defaults/main.yml
    │   ├── files/
    │   │   ├── grafana/dashboards/
    │   │   └── prometheus_main.yml
    │   ├── tasks/main.yml
    │   └── templates/
    └── target/
        ├── defaults/main.yml
        └── tasks/main.yml
```

### 🚀 Deployment Steps

1. **Configure inventory**
   ```yaml
   # inventories/hosts.yml
   all:
     hosts:
       monitoring-server:
         ansible_host: your-server-ip
         ansible_user: root
   ```

2. **Deploy monitoring stack**
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/monitoring.yml
   ```

3. **Access services**
   - **Prometheus**: `http://your-server-ip:9090`
   - **Grafana**: `http://your-server-ip:3000` (admin/admin)

### 📊 MySQL Metrics

The mysqld-exporter collects comprehensive MySQL metrics:

```bash
curl http://load-balancer-ip:9104/metrics
```

Key metrics include:
- Connection statistics
- Query performance
- Replication lag
- InnoDB status
- Buffer pool utilization

## Phase 3: Logging Stack Setup

### 🎯 Objectives

Implement ELK stack (Elasticsearch, Logstash, Kibana) for centralized log collection and analysis.

### 📝 Tasks Checklist

- [ ] Set up ELK server with appropriate resources
- [ ] Choose between Fluentd and Logstash for log processing
- [ ] Configure Ansible playbook for ELK deployment
- [ ] Verify cluster health and service status
- [ ] Configure Logstash for log separation (Auth, Syslog, Kernel)
- [ ] Create index patterns in Kibana
- [ ] Optional: Implement ILM policies

### 🔧 Deployment Steps

1. **Deploy ELK stack**
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/logging.yml
   ```

2. **Configure log pipelines**
   ```yaml
   # Logstash pipeline configuration
   input {
     beats {
       port => 5044
     }
   }
   
   filter {
     if [fields][log_type] == "mysql" {
       grok {
         match => { "message" => "%{MYSQLDLOG}" }
       }
     }
   }
   
   output {
     elasticsearch {
       hosts => ["localhost:9200"]
       index => "mysql-logs-%{+YYYY.MM.dd}"
     }
   }
   ```

3. **Access Kibana**
   ```
   http://your-server-ip:5601
   ```

## ⚙️ Configuration

### MySQL Configuration Highlights

```yaml
# values.yaml (key configurations)
auth:
  existingSecret: "mysql-credentials"
  createDatabase: true
  database: "arvan_database"

primary:
  configuration: |-
    [mysqld]
    # Performance Tuning
    innodb_buffer_pool_size=3G
    max_connections=500
    slow_query_log=1
    long_query_time=2.0

metrics:
  enabled: true
  service:
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "9104"
```

### Prometheus Configuration

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['load-balancer-ip:9104']
```

## 🔍 Monitoring & Alerts

### Key Metrics to Monitor

1. **Database Performance**
   - Query execution time
   - Connection count
   - Buffer pool hit ratio

2. **Replication Health**
   - Slave lag
   - Replication errors
   - Binary log size

3. **System Resources**
   - CPU utilization
   - Memory usage
   - Disk I/O

### Sample Grafana Dashboard

The project includes pre-configured dashboards for:
- MySQL Overview
- Replication Status
- Performance Metrics
- Resource Utilization

## 🛠️ Troubleshooting

### Common Issues

#### MySQL Connection Issues
```bash
# Check pod status
kubectl get pods -l app.kubernetes.io/name=mysql

# View logs
kubectl logs mysql-primary-0
kubectl logs mysql-secondary-0

# Test connection
kubectl exec -it mysql-primary-0 -- mysql -u root -p
```

#### Monitoring Stack Issues
```bash
# Check Prometheus targets
curl http://monitoring-server:9090/api/v1/targets

# Verify Grafana datasource
curl http://admin:admin@monitoring-server:3000/api/datasources
```

#### ELK Stack Issues
```bash
# Check Elasticsearch cluster health
curl http://elk-server:9200/_cluster/health

# View Logstash logs
docker logs logstash

# Access Kibana
curl http://elk-server:5601/api/status
```

### Performance Tuning Tips

1. **MySQL Optimization**
   - Adjust `innodb_buffer_pool_size` based on available memory
   - Configure `max_connections` according to your workload
   - Enable slow query log for performance analysis

2. **Monitoring Optimization**
   - Adjust scrape intervals in Prometheus
   - Use recording rules for complex queries
   - Set up appropriate retention policies

## 📚 Documentation

- [MySQL Configuration Guide](docs/mysql-configuration.md)
- [Monitoring Setup Guide](docs/monitoring-setup.md)
- [ELK Stack Configuration](docs/elk-configuration.md)
- [Security Best Practices](docs/security.md)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Bitnami](https://bitnami.com/) for the excellent MySQL Helm charts
- [ArvanCloud](https://arvancloud.com/) for the PaaS infrastructure
- [Prometheus Community](https://prometheus.io/) for monitoring tools
- [Elastic](https://elastic.co/) for the ELK stack

## 📞 Support

For support and questions:
- Create an issue in this repository
- Contact the maintainers
- Check the [troubleshooting guide](docs/troubleshooting.md)

---

⭐ **Star this repository if you found it helpful!**
