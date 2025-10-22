# ELK Stack Ansible Deployment

Production-ready Ansible project for deploying the complete ELK (Elasticsearch, Logstash, Kibana) Stack on RHEL-based systems.

## Overview

This project automates the installation and configuration of the ELK Stack version 8.19.5, based on tested and verified installation procedures documented in `ELK_INSTALLATION_LOG.md`. The deployment uses a role-based architecture for modularity and maintainability.

## Features

- **Automated Installation**: Complete hands-off deployment of Elasticsearch, Logstash, and Kibana
- **Credential Management**: Automatic capture and secure storage of generated passwords
- **Health Checks**: Built-in service verification and readiness checks
- **Idempotent Design**: Safe to run multiple times without side effects
- **Comprehensive Documentation**: Detailed credentials file with next steps and troubleshooting
- **Role-Based Architecture**: Modular design for easy customization and maintenance
- **Systemd Integration**: All services enabled for automatic startup on boot

## Supported Operating Systems

- Fedora 38+
- RHEL 8+
- CentOS Stream 8+
- Rocky Linux 8+
- AlmaLinux 8+

## System Requirements

- **Minimum RAM**: 2 GB (4 GB or more recommended)
- **Disk Space**: 20 GB free minimum
- **Network**: Internet access for package downloads
- **User**: sudo/root privileges required
- **Java**: OpenJDK 21 (automatically installed)

## Project Structure

```
elk-stack/
├── README.md                           # This file
├── ansible.cfg                         # Ansible configuration
├── inventory.yml                       # Inventory with variables
├── site.yml                            # Main playbook
├── ELK_INSTALLATION_LOG.md            # Reference manual installation log
├── install-elk-stack.yml              # Original working playbook
└── roles/
    ├── java_install/
    │   └── tasks/main.yml             # Java 21 installation
    ├── elasticsearch/
    │   ├── tasks/main.yml             # Elasticsearch deployment
    │   └── handlers/main.yml          # Service restart handlers
    ├── kibana/
    │   ├── tasks/main.yml             # Kibana deployment
    │   └── handlers/main.yml          # Service restart handlers
    ├── logstash/
    │   ├── tasks/main.yml             # Logstash deployment
    │   └── handlers/main.yml          # Service restart handlers
    └── credentials_management/
        └── tasks/main.yml             # Credential capture and storage
```

## Quick Start

### 1. Clone the Repository

```bash
cd /home/user/ansible/elk-stack
```

### 2. Review and Customize Inventory

Edit `inventory.yml` to configure your target hosts and variables:

```yaml
all:
  children:
    elk_servers:
      hosts:
        localhost:
          ansible_connection: local
```

### 3. Run the Playbook

For local installation:
```bash
ansible-playbook site.yml
```

For remote hosts:
```bash
ansible-playbook site.yml -i inventory.yml
```

With sudo password prompt:
```bash
ansible-playbook site.yml --ask-become-pass
```

### 4. Retrieve Credentials

After successful deployment, credentials are saved to:
- **Target machine**: `/var/elk-credentials/elk-credentials.txt`
- **Control machine**: `./elk-credentials-<hostname>.txt`

## Usage

### Running Specific Roles

Use tags to run specific components:

```bash
# Install only Java
ansible-playbook site.yml --tags java

# Install only Elasticsearch
ansible-playbook site.yml --tags elasticsearch

# Install Elasticsearch and Kibana
ansible-playbook site.yml --tags elasticsearch,kibana

# Skip credentials management
ansible-playbook site.yml --skip-tags credentials
```

### Verification

After deployment, verify the installation:

```bash
# Check service status
sudo systemctl status elasticsearch
sudo systemctl status kibana
sudo systemctl status logstash

# Test Elasticsearch (use password from credentials file)
curl -k -u elastic:YOUR_PASSWORD https://localhost:9200

# Test Kibana
curl http://localhost:5601/api/status
```

## Configuration

### Variables

Key variables are defined in `inventory.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `elastic_version` | 8.19.5 | ELK Stack version |
| `elasticsearch_port` | 9200 | Elasticsearch HTTP port |
| `kibana_port` | 5601 | Kibana web interface port |
| `logstash_port` | 5000 | Logstash input port |
| `credentials_dir` | /var/elk-credentials | Directory for credentials storage |
| `gpg_key_url` | https://artifacts.elastic.co/GPG-KEY-elasticsearch | Elastic GPG key URL |
| `elastic_repo_baseurl` | https://artifacts.elastic.co/packages/8.x/yum | Elastic repository base URL |

### Customizing Roles

Each role can be customized independently:

- **Java Install**: Modify `roles/java_install/tasks/main.yml` to change Java version
- **Elasticsearch**: Edit `roles/elasticsearch/tasks/main.yml` for custom configurations
- **Kibana**: Customize `roles/kibana/tasks/main.yml` for Kibana settings
- **Logstash**: Modify `roles/logstash/tasks/main.yml` for pipeline configurations

## Post-Installation Steps

### 1. Access Kibana

Open your browser and navigate to:
```
http://localhost:5601
```

### 2. Complete Kibana Enrollment

1. Use the enrollment token from the credentials file
2. Login with username `elastic` and the password from credentials file

### 3. Configure Logstash Pipelines

Create pipeline configuration files in `/etc/logstash/conf.d/`:

Example pipeline (`/etc/logstash/conf.d/zeek.conf`):
```ruby
input {
  file {
    path => "/opt/zeek/logs/current/*.log"
    start_position => "beginning"
  }
}

output {
  elasticsearch {
    hosts => ["https://localhost:9200"]
    user => "elastic"
    password => "YOUR_PASSWORD"
    ssl => true
    ssl_certificate_verification => false
  }
}
```

Restart Logstash after adding pipelines:
```bash
sudo systemctl restart logstash
```

## Troubleshooting

### Elasticsearch Won't Start

1. Check logs:
   ```bash
   sudo journalctl -u elasticsearch -f
   tail -f /var/log/elasticsearch/elasticsearch.log
   ```

2. Verify Java installation:
   ```bash
   java -version
   ```

3. Check system resources:
   ```bash
   free -h
   df -h
   ```

### Password Not Captured

If the Elasticsearch password wasn't automatically captured:

1. Check the Elasticsearch log:
   ```bash
   sudo grep -i "generated password" /var/log/elasticsearch/elasticsearch.log
   ```

2. Reset the password manually:
   ```bash
   sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
   ```

### Kibana Enrollment Token Missing

Generate a new enrollment token:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

### Service Management

Start/stop/restart services:
```bash
sudo systemctl start elasticsearch
sudo systemctl stop kibana
sudo systemctl restart logstash
```

Enable/disable auto-start:
```bash
sudo systemctl enable elasticsearch
sudo systemctl disable logstash
```

View service logs:
```bash
sudo journalctl -u elasticsearch -n 100 --no-pager
sudo journalctl -u kibana -f
sudo journalctl -u logstash --since "1 hour ago"
```

## Security Considerations

1. **Passwords**: Change default passwords after installation
2. **Firewall**: Configure firewall rules to restrict access
3. **SSL/TLS**: Elasticsearch uses self-signed certificates by default
4. **Credentials File**: Restrict access to credentials file (mode 0600)
5. **Network**: Consider binding services to specific interfaces

### Recommended Firewall Rules

```bash
# Allow Kibana from specific network
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="5601" protocol="tcp" accept'

# Allow Elasticsearch from localhost only (default)
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="127.0.0.1" port port="9200" protocol="tcp" accept'

# Reload firewall
sudo firewall-cmd --reload
```

## Integration with Zeek

To integrate with Zeek for network security monitoring:

1. Install Zeek on the same or different host
2. Configure Logstash to read Zeek logs
3. Create Kibana dashboards for Zeek data visualization

Example Zeek integration playbook can be added to this project.

## Development and Testing

### Testing in a Development Environment

```bash
# Dry run
ansible-playbook site.yml --check

# Verbose output
ansible-playbook site.yml -vvv

# Run on a specific host
ansible-playbook site.yml --limit localhost
```

### Extending the Project

To add new roles:

1. Create role directory:
   ```bash
   mkdir -p roles/new_role/{tasks,handlers,templates,files,vars,defaults}
   ```

2. Add role to `site.yml`:
   ```yaml
   roles:
     - role: new_role
       tags: new_role
   ```

## References

- **Elasticsearch Documentation**: https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
- **Kibana Documentation**: https://www.elastic.co/guide/en/kibana/current/index.html
- **Logstash Documentation**: https://www.elastic.co/guide/en/logstash/current/index.html
- **Ansible Documentation**: https://docs.ansible.com/
- **ELK Installation Log**: `ELK_INSTALLATION_LOG.md` (tested procedures)

## License

This project is provided as-is for educational and production use.

## Contributing

Contributions are welcome! Please ensure:
- All changes are tested on supported platforms
- Documentation is updated accordingly
- Commit messages are clear and descriptive

## Support

For issues and questions:
1. Check the `ELK_INSTALLATION_LOG.md` for tested procedures
2. Review Elasticsearch/Kibana/Logstash official documentation
3. Check system logs and service status

## Version History

- **v1.0.0**: Initial release with ELK Stack 8.19.5
  - Role-based architecture
  - Automated credential management
  - Tested on Fedora 42

## Authors

Based on manual installation testing and Ansible best practices.

---

**Note**: This deployment is based on the tested installation documented in `ELK_INSTALLATION_LOG.md` and uses proven patterns from the working `install-elk-stack.yml` playbook.
