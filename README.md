# SOC Stack Lab

## Objective

The goal of this project is to set up a comprehensive Security Operations Center (SOC) stack lab using open-source tools to enhance network monitoring and security analysis. The SOC stack will be implemented on a physical server in my home lab environment.

The core components of the stack include:

- **Wazuh:** Deployed as the primary Security Information and Event Management (SIEM) solution. Wazuh will provide real-time security monitoring, log analysis, and threat detection.
  
- **Grafana:** A separate dashboard integrated with Wazuh to visualize and analyze security metrics and logs through dynamic dashboards and graphical representations. Grafana is used due to its efficient data handling compared to Kibana.

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- **Security Information and Event Management (SIEM) system:** For log ingestion and analysis.
- **Pentesting tools:** To create realistic network traffic and attack scenarios.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Wazuh-Indexer Installation Guide

## Overview

This guide will walk you through the installation and configuration of Wazuh-Indexer, which will store and manage security logs for our SIEM stack. Wazuh-Indexer is a fork of OpenSearch, which was originally based on Elasticsearch 7.10.2.

## Table of Contents

1. [Backend Storage](#backend-storage)
2. [Log Ingestion](#log-ingestion)
3. [Installation Steps](#installation-steps)
   - [Certificates Creation](#certificates-creation)
   - [Installation](#installation)
   - [Configuration](#configuration)
   - [Memory Locking](#memory-locking)
   - [Cluster Initialization](#cluster-initialization)
4. [Wazuh-Dashboard Installation](#wazuh-dashboard-installation)

## Backend Storage

The Wazuh-Indexer will store all collected security logs, providing a scalable and reliable solution for search and analysis.

### Key Components

- **Master Node:** Manages cluster-wide settings and operations.
- **Data Node:** Stores and searches data, performing all data-related operations.
- **Ingest Node:** Pre-processes data before storing it.

**Note:** For a more detailed understanding of node types, refer to the [OpenSearch documentation](https://opensearch.org/docs/latest/opensearch/cluster/).

**High Availability:** Wazuh-Indexer nodes can be clustered to ensure high availability. For example, if a master node fails, other nodes will handle the load.

## Log Ingestion

Data in Wazuh-Indexer is organized into indices and shards. Ensure that you have at least one replica shard for high availability. 

**Disk Sizing:** Properly size your disk space based on data retention needs and log types. SSDs are recommended for production environments.

## Installation Steps

### Certificates Creation

1. Download the certificate creation script and configuration file:

    ```bash
    curl -sO https://packages.wazuh.com/4.3/wazuh-certs-tool.sh
    curl -sO https://packages.wazuh.com/4.3/config.yml
    ```

2. Edit `config.yml` to replace node names and IP addresses:

    ```yaml
    nodes:
      indexer:
        - name: node-1
          ip: <indexer-node-ip>
    ```

3. Run the certificate creation script:

    ```bash
    bash ./wazuh-certs-tool.sh -A
    tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .
    ```

### Installation

1. Install required packages:

    ```bash
    apt-get install debconf adduser procps
    apt-get install gnupg apt-transport-https
    ```

2. Import the GPG key:

    ```bash
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
    ```

3. Add the Wazuh repository:

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
    ```

4. Install Wazuh-Indexer:

    ```bash
    apt-get update
    apt-get -y install wazuh-indexer
    ```

### Configuration

1. Edit `/etc/wazuh-indexer/opensearch.yml`:

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

2. Set `bootstrap.memory_lock` and limit memory usage:

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

3. Start the service:

    ```bash
    systemctl daemon-reload
    systemctl enable wazuh-indexer
    systemctl start wazuh-indexer
    ```

### Cluster Initialization

1. Initialize the cluster:

    ```bash
    /usr/share/wazuh-indexer/bin/indexer-security-init.sh
    ```

2. Verify the cluster status:

    ```bash
    curl -k -u admin:admin https://<WAZUH_INDEXER_IP>:9200/_cat/nodes?v
    ```

## Wazuh-Dashboard Installation

The Wazuh-Dashboard provides a WebUI for interacting with the Wazuh-Indexer cluster.

### Installation

1. Install required packages:

    ```bash
    apt-get install debhelper tar curl libcap2-bin
    ```

2. Import the GPG key:

    ```bash
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
    ```

3. Add the repository:

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
    ```

4. Install Wazuh-Dashboard:

    ```bash
    apt-get update
    apt-get -y install wazuh-dashboard
    ```

### Configuration

1. Edit `/etc/wazuh-dashboard/opensearch_dashboards.yml`:

    ```yaml
    server.host: 0.0.0.0
    server.port: 443
    opensearch.hosts: ["https://localhost:9200"]
    opensearch.ssl.verificationMode: certificate
    ```

2. Deploy certificates:

    ```bash
    NODE_NAME=<dashboard-node-name>
    ```

3. Start the service:

    ```bash
    systemctl daemon-reload
    systemctl enable wazuh-dashboard
    systemctl start wazuh-dashboard
    ```

4. Configure Wazuh-Dashboard:

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

5. Secure the installation:

    ```bash
    /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
    echo <kibanaserver-password> | /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore --allow-root add -f --stdin opensearch.password
    systemctl restart wazuh-dashboard
    ```

## Conclusion

This guide covers the installation and configuration of Wazuh-Indexer and Wazuh-Dashboard. With these components, you will have a scalable backend storage solution for managing and analyzing security logs.
