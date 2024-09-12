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

# Project Steps
### Setting Up Wazuh

**1. Preparation**

- **Update and Upgrade System:** Ensure the server’s package list and software are up-to-date.
- **Install Dependencies:** Install any required dependencies for Wazuh components.

**2. Install Wazuh Manager**

- **Add Wazuh Repository:** Add the Wazuh GPG key and repository to the server.
- **Install Wazuh Manager:** Use the package manager to install the Wazuh Manager.
- **Configure Wazuh Manager:** Edit the configuration files (typically found in `/var/ossec/etc/ossec.conf`) to set up basic parameters and ensure the Manager is set to listen on the correct network interfaces and ports.
- **Start Wazuh Manager:** Enable and start the Wazuh Manager service.

**3. Install Wazuh Indexer**

- **Install Wazuh Indexer:** Use the package manager to install the Wazuh Indexer.
- **Configure Wazuh Indexer:** Edit the configuration files (usually located in `/etc/wazuh-indexer/`). Ensure that the Indexer is configured to communicate with the Wazuh Manager and adjust settings to bind to the appropriate interfaces.
- **Start Wazuh Indexer:** Enable and start the Wazuh Indexer service.

**4. Install Wazuh Dashboard**

- **Install Wazuh Dashboard:** Use the package manager to install the Wazuh Dashboard.
- **Configure Wazuh Dashboard:** Edit the configuration files (found in `/etc/wazuh-dashboard/`) to map the Dashboard to the Wazuh Manager and Indexer. This includes setting the correct URLs and authentication details so that the Dashboard can communicate with the other Wazuh components.
- **Start Wazuh Dashboard:** Enable and start the Wazuh Dashboard service.

**5. Configuration Adjustments**

- **Service Mapping:** Ensure that all services (Manager, Indexer, Dashboard) are correctly mapped to the server’s internal IP address or hostname. Update configuration files to reflect that all components are hosted on the same server.
- **Network and Port Settings:** Verify that the ports used by Wazuh components are open and correctly configured in the firewall and networking settings.
- **Verify Connections:** Ensure that the Wazuh Manager, Indexer, and Dashboard can communicate with each other correctly. Test and confirm that logs and metrics are flowing properly between the services.

### Setting Up Grafana

**1. Configure Wazuh for Grafana Integration**

- **Verify Wazuh Indexer:** Ensure that the Wazuh Indexer is properly configured and running, as Grafana will query data from it.
- **Access Wazuh API:** Confirm that the Wazuh API is accessible and properly configured to allow Grafana to pull data. The API endpoint is typically used for querying data.

**2. Add Wazuh as a Data Source in Grafana**

- **Navigate to Configuration > Data Sources.**
- **Click on Add data source.**
- **Select Wazuh or Elasticsearch** (if Wazuh uses Elasticsearch as the backend).
- **Configure Data Source:**
  - **URL:** Enter the URL of the Wazuh API or Elasticsearch instance. This is where Grafana will send queries to.
  - **Index Name:** Specify the index pattern if you are using Elasticsearch.
  - **Authentication:** Set up any required authentication methods if needed (e.g., API keys or credentials).
  - **Test Connection:** Click Save & Test to verify that Grafana can connect to the Wazuh Indexer or Elasticsearch.

**3. Create Grafana Dashboards**

- **Create a New Dashboard:**
  - Go to the + icon in the sidebar and select Dashboard.
  - Click Add new panel to start building visualizations.
- **Configure Panels:**
  - **Query Data:** Use the query editor to fetch data from Wazuh or Elasticsearch. You can create queries to display logs, alerts, and other security metrics.
  - **Visualizations:** Choose the type of visualizations you want (graphs, tables, charts) based on the data you are querying.
  - **Customize Panels:** Configure the appearance and settings of each panel according to your needs.
