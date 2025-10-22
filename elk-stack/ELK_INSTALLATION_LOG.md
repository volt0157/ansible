# ELK Stack Installation on Fedora 42 - Step by Step Log

## Environment
- OS: Fedora 42 (Adams)
- Kernel: 6.16.12-200.fc42.x86_64
- RAM: 15Gi available
- Disk: 232G total, 111G free
- User: ahead (with sudo access)
- Package Manager: dnf5

## Completed Steps

### Step 1: Install Java 21
**Command:**
```
sudo dnf install -y java-21-openjdk java-21-openjdk-devel
```
**Status:** SUCCESS
**Verification:**
```
java -version
openjdk version "21.0.8" 2025-07-15
OpenJDK Runtime Environment (Red_Hat-21.0.8.0.9-1) (build 21.0.8+9)
OpenJDK 64-Bit Server VM (Red_Hat-21.0.8.0.9-1) (build 21.0.8+9, mixed mode, sharing)
```

### Step 2: Import Elasticsearch GPG Key
**Command:**
```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
**Status:** SUCCESS

### Step 3: Add Elasticsearch Repository (Fedora 42 with dnf5)
**Command:**
```
cat <<EOF | sudo tee /etc/yum.repos.d/elasticsearch.repo
[elastic-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```
**Status:** SUCCESS
**Note:** dnf config-manager --add-repo URL method failed with 404 on dnf5. Manual repo file creation worked.

### Step 4: Install Elasticsearch
**Command:**
```
sudo dnf install -y elasticsearch
```
**Status:** SUCCESS
**Version:** 8.19.5-1
**Auto-generated elastic password:** LgUNjfpRtRTQqR*R=aRh

### Step 5: Enable and Start Elasticsearch
**Commands:**
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```
**Status:** SUCCESS

### Step 6: Create Kibana Repository
**Command:**
```
cat <<EOF | sudo tee /etc/yum.repos.d/kibana.repo
[kibana-8.x]
name=Kibana repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```
**Status:** SUCCESS

### Step 7: Install Kibana
**Command:**
```
sudo dnf install -y kibana
```
**Status:** SUCCESS
**Version:** 8.19.5-1

### Step 8: Enable and Start Kibana
**Commands:**
```
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```
**Status:** SUCCESS

### Step 9: Create Logstash Repository
**Command:**
```
cat <<EOF | sudo tee /etc/yum.repos.d/logstash.repo
[logstash-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```
**Status:** SUCCESS

### Step 10: Install Logstash
**Command:**
```
sudo dnf install -y logstash
```
**Status:** SUCCESS
**Version:** 8.19.5-1

### Step 11: Enable and Start Logstash
**Commands:**
```
sudo systemctl daemon-reload
sudo systemctl enable logstash.service
sudo systemctl start logstash.service
```
**Status:** SUCCESS

---

## ELK Stack Status Summary
- Elasticsearch: Running on https://localhost:9200
- Kibana: Running on http://localhost:5601
- Logstash: Running (awaiting pipeline configuration)
- All versions: 8.19.5

## Credentials
- Elasticsearch elastic user password: LgUNjfpRtRTQqR*R=aRh

## Notes for Ansible Conversion
- Use dnf module with state=present for package installations
- GPG key import can use rpm_key module or bash with rpm command
- Repository file creation can use template module or copy module
- Need to handle dnf5 differences (no --add-repo flag)
- Capture Elasticsearch auto-generated password using shell module and register
- Version consistency: All ELK components must use same version (8.x in this case)
- Use systemctl module for enabling and starting services
- Consider using handlers for service restarts

