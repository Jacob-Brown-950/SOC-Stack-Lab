# SOC Stack Lab

## Objective

The goal of this project is to set up a comprehensive Security Operations Center (SOC) stack lab using open-source tools to enhance network monitoring and security analysis. The SOC stack will be implemented on a physical server in my home lab environment. There is a much easier way to do this now, but I chose the older hands on way to refine my understanding of nodes.

The core components of the stack include:

- **Wazuh Indexer:** Backend storage for ingested logs
- **Wazuh Manager:** Log cleanup and formatting, receives logs from the deployed agents
- **Wazuh Dashboard:** Central visualization for data collected

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- **Security Information and Event Management (SIEM) system:** For log ingestion and analysis.
- **Pentesting tools:** To create realistic network traffic and attack scenarios.

## Table of Contents

1. [Introduction](#introduction)
2. [Wazuh-Indexer Installation](#wazuh-indexer-installation)
    - [Certificates Creation](#certificates-creation)
    - [Installation](#installation)
    - [Configuration](#configuration)
    - [Cluster Initialization](#cluster-initialization)
3. [Wazuh-Dashboard Installation](#wazuh-dashboard-installation)
    - [Installation](#installation-1)
    - [Configuration](#configuration-1)
4. [Wazuh-Manager Installation](#wazuh-manager-installation)
    - [Installation](#installation-2)
    - [Configuration](#configuration-2)
    - [Verification](#verification-2)
5. [Deploying Wazuh Agent](#deploying-wazuh-agent)
    - [Installation](#installation-3)
    - [Configuration](#configuration-3)
    - [Simulating Alerts](#simulating-alerts)
6. [Final Configuration](#final-configuration)
7. [Troubleshooting](#troubleshooting)

## Introduction

This guide outlines the installation of Wazuh components on Ubuntu 22.04. We will install and configure the Wazuh-Indexer, Wazuh-Dashboard, and Wazuh-Manager.

### Key Components

- **Master Node:** Manages cluster-wide settings and operations.
- **Data Node:** Stores and searches data, performing all data-related operations.
- **Ingest Node:** Pre-processes data before storing it.

**Note:** For a more detailed understanding of node types, refer to the [OpenSearch documentation](https://opensearch.org/docs/latest/opensearch/cluster/).

**High Availability:** Wazuh-Indexer nodes can be clustered to ensure high availability. For example, if a master node fails, other nodes will handle the load.

## Wazuh-Indexer Installation

### Certificates Creation

1. **Download the certificate creation script and configuration file:**

    ```bash
    curl -sO https://packages.wazuh.com/4.3/wazuh-certs-tool.sh
    curl -sO https://packages.wazuh.com/4.3/config.yml
    ```

2. **Edit `config.yml` to replace node names and IP addresses:**

    ```yaml
    nodes:
      indexer:
        - name: node-1
          ip: <indexer-node-ip>
    ```

3. **Run the certificate creation script:**

    ```bash
    bash ./wazuh-certs-tool.sh -A
    tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .
    ```

### Installation

1. **Install required packages:**

    ```bash
    apt-get install debconf adduser procps
    apt-get install gnupg apt-transport-https
    ```

2. **Import the GPG key:**

    ```bash
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
    ```

3. **Add the Wazuh repository:**

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
    ```

4. **Install Wazuh-Indexer:**

    ```bash
    apt-get update
    apt-get -y install wazuh-indexer
    ```

### Configuration

1. **Edit `/etc/wazuh-indexer/opensearch.yml`:**

    ```yaml
    network.host: <node-address>
    node.name: <node-name>
    cluster.initial_master_nodes:
      - "node-1"
      - "node-2"
      - "node-3"
    discovery.seed_hosts:
      - "10.0.0.1"
      - "10.0.0.2"
      - "10.0.0.3"
    plugins.security.nodes_dn:
      - "CN=node-1,OU=Wazuh,O=Wazuh,L=California,C=US"
      - "CN=node-2,OU=Wazuh,O=Wazuh,L=California,C=US"
      - "CN=node-3,OU=Wazuh,O=Wazuh,L=California,C=US"
    ```

2. **Set `bootstrap.memory_lock` and limit memory usage:**

    ```bash
    nano /etc/wazuh-indexer/opensearch.yml
    # Add the line
    bootstrap.memory_lock: true

    nano /usr/lib/systemd/system/wazuh-indexer.service
    # Add the line
    LimitMEMLOCK=infinity

    nano /etc/wazuh-indexer/jvm.options
    # Set heap size
    -Xms4g
    -Xmx4g
    ```

3. **Start the service:**

    ```bash
    systemctl daemon-reload
    systemctl enable wazuh-indexer
    systemctl start wazuh-indexer
    ```

### Cluster Initialization

1. **Initialize the cluster:**

    ```bash
    /usr/share/wazuh-indexer/bin/indexer-security-init.sh
    ```

2. **Verify the cluster status:**

    ```bash
    curl -k -u admin:admin https://<WAZUH_INDEXER_IP>:9200/_cat/nodes?v
    ```

## Wazuh-Dashboard Installation

The Wazuh-Dashboard provides a WebUI for interacting with the Wazuh-Indexer cluster.

### Installation

1. **Install required packages:**

    ```bash
    apt-get install debhelper tar curl libcap2-bin
    ```

2. **Import the GPG key:**

    ```bash
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
    ```

3. **Add the repository:**

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
    ```

4. **Install Wazuh-Dashboard:**

    ```bash
    apt-get update
    apt-get -y install wazuh-dashboard
    ```

### Configuration

1. **Edit `/etc/wazuh-dashboard/opensearch_dashboards.yml`:**

    ```yaml
    server.host: 0.0.0.0
    server.port: 443
    opensearch.hosts: ["https://localhost:9200"]
    opensearch.ssl.verificationMode: certificate
    ```

2. **Deploy certificates:**

    ```bash
    NODE_NAME=<dashboard-node-name>
    ```

3. **Start the service:**

    ```bash
    systemctl daemon-reload
    systemctl enable wazuh-dashboard
    systemctl start wazuh-dashboard
    ```

4. **Configure Wazuh-Dashboard:**

    ```bash
    nano /usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml
    # Replace with Wazuh server IP or hostname
    hosts:
      - default:
        url: https://localhost
        port: 55000
        username: wazuh-wui
        password: wazuh-wui
        run_as: false
    ```

5. **Secure the installation:**

    ```bash
    /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
    echo <kibanaserver-password> | /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore --allow-root add -f --stdin opensearch.password
    systemctl restart wazuh-dashboard
    ```

## Wazuh-Manager Installation

### Installation

1. **Install Wazuh-Manager:**

    ```bash
    apt-get update
    apt-get install wazuh-manager
    ```

### Configuration

1. **Configure Wazuh-Manager:**

    Edit `/var/ossec/etc/ossec.conf` to include the necessary settings for your environment. This typically includes configuring the `wazuh` and `rules` sections.

2. **Enable Password Authentication:**

    Add the following to the `<auth>` section of `/var/ossec/etc/ossec.conf`:

    ```xml
    <auth>
      <use_password>yes</use_password>
    </auth>
    ```

    Set the agent enrollment password:

    ```bash
    echo "<CUSTOM_PASSWORD>" > /var/ossec/etc/authd.pass
    chmod 640 /var/ossec/etc/authd.pass
    chown root:wazuh /var/ossec/etc/authd.pass
    ```

3. **Enable Vulnerability Detection:**

    Edit `/var/ossec/etc/ossec.conf` to enable vulnerability detection.

### Verification

1. **Start the Wazuh-Manager Service:**

    ```bash
    systemctl daemon-reload
    systemctl enable wazuh-manager
    systemctl start wazuh-manager
    ```

2. **Verify the Installation:**

    ```bash
    /var/ossec/bin/ossec-control status
    ```

## Deploying Wazuh Agent

Deploying the Wazuh agent on client machines is crucial for log collection and threat detection. Below is a brief overview of the deployment process and the activities performed to generate alerts.

### Installation

1. **Generate and Deploy the Wazuh Agent Installation Script:**
   - For Mac, Linux, and Windows it's all the same. Generate a script for the appropriate OS and run it in as an administrator on the endpoint.

### Simulating Alerts

1. **Simulate Failed SSH Login Attempts:**
   - Use Hydra to perform brute-force attacks and generate failed SSH login attempt logs.

2. **Generate Port Scanning Logs:**
   - Utilize Nmap to scan ports on the target systems, creating additional log entries.

3. **Assess Cybersecurity Hygiene:**
   - Review the assessments and recommendations provided by the Wazuh agent. For example, the agent may highlight weak practices such as using default account names like "admin" or "user", which could give attackers an advantage.

By deploying the Wazuh agent and simulating various activities, you can verify the effectiveness of your security setup and improve your overall cybersecurity posture based on the agent's recommendations.

## Final Configuration

1. **Ensure Integration:**

    - **Wazuh-Manager** communicates with **Wazuh-Indexer** and **Wazuh-Dashboard**.
    - Configure data flows and access permissions.

2. **Secure Communications:**

    - Ensure all communications between nodes and services are encrypted.
    - Verify that SSL/TLS certificates are correctly deployed.

## Troubleshooting

- **Wazuh-Indexer Not Starting:**
  - Check logs for errors: `/var/log/wazuh-indexer.log`.
  - Verify configurations in `/etc/wazuh-indexer/opensearch.yml`.

- **Wazuh-Dashboard Not Accessible:**
  - Check service status: `systemctl status wazuh-dashboard`.
  - Verify network settings and firewall rules.

- **Agent Communication Issues:**
  - Check agent logs: `/var/ossec/logs/ossec.log`.
  - Ensure that Wazuh-Manager and Wazuh-Dashboard are reachable.

This guide covers the essential steps for setting up Wazuh components and deploying the Wazuh agent. Refer to the Wazuh documentation for advanced configurations and additional details.
