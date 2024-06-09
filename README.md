
# RKE2 Ansible Setup

This Ansible project automates the deployment of a highly available RKE2 cluster with two master nodes and three worker nodes. It includes configurations for Docker, RKE2, and Keepalived to ensure high availability.

## Directory Structure

```
rke2-ansible-updated/
├── ansible.cfg
├── inventory
│   └── hosts.ini
├── playbooks
│   ├── install_rke2.yaml
│   └── roles
│       ├── common
│       │   ├── tasks
│       │   │   └── main.yaml
│       │   └── handlers
│       │       └── main.yaml
│       ├── master
│       │   └── tasks
│       │       └── main.yaml
│       ├── worker
│       │   └── tasks
│       │       └── main.yaml
│       └── keepalived
│           └── tasks
│               └── main.yaml
└── templates
    ├── docker_daemon.json.j2
    ├── rke2_master_config.yaml.j2
    ├── rke2_worker_config.yaml.j2
    └── keepalived.conf.j2
```

## Prerequisites

- Ansible installed on your control machine.
- SSH access to all nodes (masters and workers).
- The private registry must be set up and accessible.

## Inventory Configuration

Update the `inventory/hosts.ini` file with your actual hostnames and variables.

```ini
[masters]
master1 ansible_host=master1.hpc.local
master2 ansible_host=master2.hpc.local

[workers]
worker1 ansible_host=worker1.hpc.local
worker2 ansible_host=worker2.hpc.local
worker3 ansible_host=worker3.hpc.local

[all:vars]
ansible_python_interpreter=/usr/bin/python3
private_registry="repo.hpc.local:5000"
rke2_server_url="https://rke2-master.hpc.local:9345"
rke2_token="YOUR_CLUSTER_TOKEN"
secondary_dns="172.16.0.1"
```

## Usage

### Step 1: Update Configuration

1. Replace `YOUR_CLUSTER_TOKEN` in the `inventory/hosts.ini` file with the actual token from your RKE2 master node.

### Step 2: Run the Playbook

Run the Ansible playbook to set up the cluster:

```bash
ansible-playbook playbooks/install_rke2.yaml
```

### Keepalived Configuration

The `keepalived.conf.j2` template includes a basic configuration for Keepalived. Update the `auth_pass` and `virtual_ipaddress` values as needed.

```conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass your_password
    }
    virtual_ipaddress {
        192.168.66.20
    }
}
```

### Network Configuration

The `docker_daemon.json.j2` template configures Docker to use the local registry.

```json
{
  "insecure-registries" : ["{{ private_registry }}"]
}
```

### RKE2 Configuration

The `rke2_master_config.yaml.j2` and `rke2_worker_config.yaml.j2` templates set up the RKE2 configurations for masters and workers, respectively.

#### Master Configuration

```yaml
private-registry:
  mirrors:
    "docker.io":
      endpoint:
        - "http://{{ private_registry }}"
resolv-conf: /etc/rancher/rke2/resolv.conf
```

#### Worker Configuration

```yaml
token: "{{ rke2_token }}"
server: "{{ rke2_server_url }}"
private-registry:
  mirrors:
    "docker.io":
      endpoint:
        - "http://{{ private_registry }}"
resolv-conf: /etc/rancher/rke2/resolv.conf
```

## Troubleshooting

If you encounter issues, check the logs in `/var/log/rke2-setup.log` on each node.

### Checking Services

For masters:

```bash
systemctl status rke2-server
```

For workers:

```bash
systemctl status rke2-agent
```

### Reload Systemd and Restart Services

If you make changes to the systemd service files, reload systemd and restart the services:

```bash
sudo systemctl daemon-reload
sudo systemctl restart rke2-server
sudo systemctl restart rke2-agent
```

## License

This project is licensed under the MIT License.
