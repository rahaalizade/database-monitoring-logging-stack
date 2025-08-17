# MySQL Master-Slave Cluster with Monitoring & Logging

A comprehensive deployment guide for setting up a MySQL cluster with integrated monitoring and logging capabilities using ArvanCloud PaaS, Kubernetes, and modern observability tools.

## 🏬 Architecture Overview

This project implements a development-ready MySQL infrastructure with:

- **MySQL Master-Slave Cluster**: High availability database setup
- **Prometheus & Grafana**: Real-time monitoring and visualization
- **ELK Stack**: Centralized logging and log analysis
- **ArvanCloud PaaS**: Kubernetes-based container orchestration for mysql cluster set-up

![MySql-Cluster-design.png](MySql-Cluster-design.png)

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
- **Security**: Proper secret management like ansible-vault
- **Scalability**: Kubernetes-native deployment for easy scaling
- **Automation**: Ansible playbooks for infrastructure as code

## 📦 Prerequisites

### System Requirements

**Per MySQL Instance:**
- 2 CPU cores
- 4 GB RAM
- 20 GB SSD storage

**Monitoring Server:**
-  CPU cores
- 4 GB RAM
- 30-50 GB storage

**Logging Server (ELK):**
- 6 or 8 CPU cores
- 12 GB RAM
- 30-50 GB storage

### Tools & Dependencies

- **Kubernetes Cluster** (ArvanCloud PaaS)
- **Helm** (v3.x)
- **kubectl** configured with cluster access
- **Ansible** 
- **Docker** 

## 🏛️ Architecture

The project follows a three-tier architecture:

1. **Data Layer**: MySQL Master-Slave cluster 
2. **Monitoring Layer**: Prometheus for metrics collection, Grafana for visualization
3. **Logging Layer**: Elasticsearch for storage, Logstash for processing, Kibana for analysis and filebeat for sending auth, syslog and kernel logs from the host to logstash

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   MySQL Master  │────│  MySQL Slave    │    │   Load Balancer │
│   (Primary)     │    │  (Secondary)    │────│   (Public IP)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
┌─────────────────┐    ┌─────────────────┐    ┌───────────────────────────┐
│   Prometheus    │────│    Grafana      │    │   ELK Stack With filebeat |
│   (Metrics)     │    │  (Dashboard)    │    │   (Logging)               |
└─────────────────┘    └─────────────────┘    └───────────────────────────┘
```

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/mysql-cluster-monitoring.git
   and you will see three 2 ansible playbooks and one helm chart
   ```

1.5. **Attention**
   ```bash
   before everything SET IP OF YOUR ELASTIC AND MONITORING STACK in these 3 files!
   	elk-stack/inventory/group_vars/all.yml
	   elk-stack/inventory/hosts.yml
	   monitoring-stack/inventory/hosts.yml
   ```

2. **Configure kubectl**
   ```bash
   # Download and configure your kubeconfig from ArvanCloud
   export KUBECONFIG=/path/to/your/kubeconfig
   kubectl get pods -n <the namespace you provided when setting up the mysql cluster on PaaS environment>
   ```

3. **Add Bitnami Helm repository (This is for initial setup on PaaS)**
   ```bash
   After helm repo installed in PaaS environment try to pull it with this command to easily work on it \!
   helm pull mysql/mysql --untar

   ```

### 📊 MySQL Metrics

The mysqld-exporter collects comprehensive MySQL metrics:

```bash
curl http://<load-balancer-ip>:9104/metrics
```

Key metrics include:
- Connection statistics
- Query performance
- Replication lag
- InnoDB status
- Buffer pool utilization
- slow queries


4. **Deploy the complete stack**
   ```bash
   # Phase 1: MySQL Cluster
   First of all check your kubernetes context
   helm upgrade --install mysql mysql -f values.yaml ./
   
   # Phase 2: Monitoring
   First of all try to ping your hosts to make sure that your ansible has been properly connected with:
   ansible -i inventory/hosts.yml -m ping all
   Then install it with: 
   ansible-playbook -i inventory/hosts.yml deploy.yml --vault-password-file=.vault_pass --become -i <Identity File>
   
   # Phase 3: Logging
   Install it using ansible via:
   ansible-playbook -i inventory/hosts.yml deploy.yml --become -i <Identity File>
   ```
   This will install elk with filebeat on the server that you want to be your ELK-Stack server, in addition it will install filebeat as a docker container on your other server which will be your Monitoring-Stack

## Phase 1: MySQL Cluster Deployment

### 🎯 Objectives

Deploy a Kubernetes-based MySQL primary-secondary cluster using Bitnami's Helm chart with:
- Automated replication setup
- Performance tuning (Optional)
- Security configurations (Using kubernetes sercret manager)
- Metrics exposure with the help of mysqld-exporter on mysql pods (mysqld exporter as a container in the pod)
- Automated backups with scheduled time (Using cronjobs in kubernetes)

### 📝 Tasks Checklist

- [ ] Set up kubeconfig on local machine
- [ ] Create MySQL Helm chart configuration
- [ ] Create `mysql-credentials` secret
- [ ] Install Helm chart with custom values
- [ ] Configure mysqld-exporter for metrics
- [ ] Verify container health and database connectivity
- [ ] Set up load balancer for public access
- [ ] Create a Manifest for Cronjob for create backup of the database 

### 🔧 Deployment Steps

1. **Create the MySQL credentials secret with commandline or using Arvan UI (simple way)**
   ```bash
   kubectl create secret generic mysql-credentials \
     --from-literal=mysql-root-password='your-root-password' \
     --from-literal=mysql-password='your-user-password' \
     --from-literal=mysql-replication-password='your-replication-password'
   ```

2. **Deploy MySQL cluster**
   ```bash
   helm install mysql mysql/mysql -f values.yaml
   ```

3. **Verify deployment**
   ```bash
   kubectl get pods -n <NAMESPACE> -l app.kubernetes.io/name=mysql
   kubectl logs mysql-primary-0
   kubectl logs mysql-secondary-0
   ```

### 🔧 Automate MYSQL backup Steps

1. **Create the cronjob with the volume in order to backup from the database on schedule**
   ```bash
   kubectl apply -f cronjob-mysqldump.yml
   ```

2. **For testing it instantly, create a job from it**
   ```bash
   kubectl create job test-db-backup --from=cronjob/mysql-backup
   ```

3. **Verify Backup with another pod mounted with that Volume**
   ```bash
   kubectl run backup-browser --image=busybox --rm -it \
   --overrides='{"spec":{"containers":[{"name":"backup-browser","image":"busybox","volumeMounts":[{"name":"backup-storage","mountPath":"/backup"}],"stdin":true,"tty":true}],"volumes":[{"name":"backup-storage","persistentVolumeClaim":{"claimName":"mysql-backup-pvc"}}]}}' \
   -- sh

   / # ls backup/
   backup_20250815_201421.sql
   ```


### 🔗 Load Balancer Configuration

Configure public IP allocation for mysql-primary and mysql-secondary in ArvanCloud dashboard:
- **Ports**: 9104 (Metrics)
- **Protocol**: TCP
- **Type**: Load Balancer

## Phase 2: Monitoring Stack Setup

### 🎯 Objectives

Deploy Prometheus & Grafana stack using Ansible to monitor MySQL performance and cluster health of monitoring and elastic stacks.

### 🚀 Deployment Steps

0. **Configure credentials**

   Try to create it with this command(just the first time):

   ```bash
   ansible-vault create roles/observer/vars/secrets.yml
   ```

   with this format in it ==> vault_VARIABLE_NAME: VARIABLE_VALUE
   for reducing complexity define a variable named .vault_pass for your vault password and pass it with your command


1. **Configure inventory**
   ```yaml
   # inventory/hosts.yml
   all:
     hosts:
       monitoring-server:
         ansible_host: your-server-ip
         ansible_user: root or your desired user
   ```

2. **Deploy monitoring stack**
   ```bash
   ansible-playbook -i inventory/hosts.yml deploy.yml --become --vault-password-file=.vault_pass 
   ```

3. **Access services**
   - **Prometheus**: `http://your-server-ip:9090`
   - **Grafana**: `http://your-server-ip:3000`

## Phase 3: Logging Stack Setup

### 🎯 Objectives

Implement ELK stack (Elasticsearch, Logstash, Kibana, Filebeat) for centralized log collection and analysis.

### 📝 Tasks Checklist

- [ ] Set up ELK server with appropriate resources
- [ ] Choose between Fluentd and Logstash for log processing
- [ ] Configure Ansible playbook for ELK deployment
- [ ] Verify cluster health and service status
- [ ] Configure Logstash for log separation (Auth, Syslog, Kernel)
- [ ] Create index patterns in Kibana
- [ ] Create dashboards for different indices in Kibana
- [ ] Optional: Implement ILM policies

### 🔧 Deployment Steps

1. **Deploy ELK stack**
   ```bash
   ansible-playbook -i inventory/hosts.yml deploy.yml --become
   ```

2. **Configure log pipelines**
   ```yaml
   # Logstash pipeline configuration
      input {
         beats { port => 5044 }
         }

         filter {
         mutate {
            add_field => { "processed_at" => "%{+YYYY-MM-dd'T'HH:mm:ssZ}" } 
         }
         date {
            match => ["processed_at", "ISO8601"]
            target => "processed_at"
         }
         }

         output {
         if [log_type] == "auth" {
            elasticsearch {
               hosts => ["elasticsearch:9200"]
               data_stream => "true"
               data_stream_type => "logs"
               data_stream_dataset => "auth"
               data_stream_namespace => "default"
            }
         }
         else if [log_type] == "kernel" {

      ... and so for syslog and kernel logs
      }

   ```

3. **Access Kibana**
   ```
   http://your-server-ip:5609
   ```
4. **Access Elasticsearch and its Health**
   ```
   http://your-server-ip:9209
   http://your-server-ip:9209/_cluster/health
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
    innodb_buffer_pool_size => 70% of RAM
    max_connections => adjust it with your needs and load of your database
    slow_query_log=1
    long_query_time=2.0

metrics:
  enabled: true
  service:
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "9104"
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
- mysqld Exporter and MySQL dashboards
- MySQL replication Overview

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
curl http://elk-server:9209/_cluster/health

# View Logstash logs
docker logs logstash

# Access Kibana
curl http://elk-server:5609/api/status
```

### Performance Tuning Tips

1. **MySQL Optimization**
   - Adjust `innodb_buffer_pool_size` based on available memory
   - Configure `max_connections` according to your workload
   - Enable slow query log for performance analysis

2. **Monitoring Optimization**
   - Adjust scrape intervals in Prometheus

## 📚 Documentation

- [MySQL Configuration Guide](docs/mysql-configuration.md)
- [Monitoring Setup Guide](docs/monitoring-setup.md)
- [ELK Stack Configuration](docs/elk-configuration.md)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Bitnami](https://bitnami.com/) for the excellent MySQL Helm charts
- [ArvanCloud](https://arvancloud.com/) for the PaaS infrastructure
- [Prometheus Community](https://prometheus.io/) for monitoring tools
- [Elastic](https://elastic.co/) for the ELK stack

---