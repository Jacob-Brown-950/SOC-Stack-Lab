# SOC Stack Lab

## Objective

The goal of this project is to set up a comprehensive Security Operations Center (SOC) stack lab using open-source tools to enhance network monitoring and security analysis. The SOC stack will be implemented on a physical server in my home lab environment.

The core components of the stack include:

Wazuh: Deployed as the primary Security Information and Event Management (SIEM) solution, Wazuh will provide real-time security monitoring, log analysis, and threat detection.

Grafana: Separate Dashboard Integrated with Wazuh to visualize and analyze security metrics and logs through dynamic dashboards and graphical representations.

### Skills Learned
[Bullet Points - Remove this afterwards]

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Pentesting tools to create realistic network traffic and attack scenarios.

## Steps

1. Preparation
Update and Upgrade System: Ensure the server’s package list and software are up-to-date.
Install Dependencies: Install any required dependencies for Wazuh components.

2. Install Wazuh Manager
Add Wazuh Repository: Add the Wazuh GPG key and repository to the server.
Install Wazuh Manager: Use the package manager to install the Wazuh Manager.
Configure Wazuh Manager: Edit the configuration files (typically found in /var/ossec/etc/ossec.conf) to set up basic parameters and ensure the Manager is set to listen on the correct network interfaces and ports.
Start Wazuh Manager: Enable and start the Wazuh Manager service.

3. Install Wazuh Indexer
Install Wazuh Indexer: Use the package manager to install the Wazuh Indexer.
Configure Wazuh Indexer: Edit the configuration files to set up the Indexer. The configuration files are usually located in /etc/wazuh-indexer/. Ensure that the Indexer is configured to communicate with the Wazuh Manager and adjust settings to bind to the appropriate interfaces.
Start Wazuh Indexer: Enable and start the Wazuh Indexer service.

4. Install Wazuh Dashboard
Install Wazuh Dashboard: Use the package manager to install the Wazuh Dashboard.
Configure Wazuh Dashboard: Edit the configuration files (found in /etc/wazuh-dashboard/) to map the Dashboard to the Wazuh Manager and Indexer. This includes setting the correct URLs and authentication details so that the Dashboard can communicate with the other Wazuh components.
Start Wazuh Dashboard: Enable and start the Wazuh Dashboard service.

5. Configuration Adjustments
Service Mapping: Ensure that all services (Manager, Indexer, Dashboard) are correctly mapped to the server’s internal IP address or hostname. Update configuration files to reflect that all components are hosted on the same server.
Network and Port Settings: Verify that the ports used by Wazuh components are open and correctly configured in the firewall and networking settings.
Verify Connections: Ensure that the Wazuh Manager, Indexer, and Dashboard can communicate with each other correctly. Test and confirm that logs and metrics are flowing properly between the services.

