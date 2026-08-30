# Multicloud SOC (Azure + AWS + GCP + On-Premise)
*Implementation of a Centralized Multi-cloud Security Operations Center (SOC) using Microsoft Sentinel (SIEM + SOAR) and Microsoft Defender for Cloud (CSPM + CWPP).*

---

## 📌 Project Overview
This project documents the architecture and implementation of a centralized **Multicloud SOC** hosted in **Microsoft Azure**, integrating security visibility, posture management, and threat telemetry from **Amazon Web Services (AWS)**, **Google Cloud Platform (GCP)**, and **On-Premise** environments.

The solution centralizes audit logs and security events into a dedicated **Log Analytics Workspace** and leverages **Microsoft Sentinel** alongside **Microsoft Defender for Cloud (MDC)** for event correlation, anomaly detection, and automated incident response (SOAR).

---

## 📊 Solution Architecture
The following diagram illustrates the overall data flow, connectors, and automated response capabilities across the different clouds and the central Azure SOC:

![Multicloud SOC Architecture](https://github.com/victormatos-sec/cloud-portafolio/blob/main/PROJECT-SOC-MULTICLOUD/Diagram.jpg)

### Key Benefits:
* **Unified Visibility:** Centralized monitoring across multi-cloud and on-premise workloads.
* **Early Threat Detection:** Advanced analytical rules running against API and audit logs.
* **Automated and Orchestrated Response (SOAR):** Playbooks via Azure Logic Apps for rapid mitigation and notification.
* **Compliance and Governance:** Continuous security posture assessment (CSPM).
* **Risk and Operational Cost Reduction:** Streamlined security incident management.

---

## 🛠️ Step 1: Initial Azure Setup (Log Analytics & Sentinel)
1. Created a dedicated **Log Analytics Workspace** for centralized log storage and analysis.
2. Deployed and initialized an instance of **Microsoft Sentinel** on top of the workspace.
3. Verified access to the Sentinel Content Hub to manage native security solutions and integrations.

![Sentinel Configuration](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-content-hub.png)

---

## ☁️ Step 2: Amazon Web Services (AWS) Integration

### 2.1 Enabling Defender for Cloud (CSPM)
* Verified that the foundational **Microsoft Defender for Cloud (CSPM)** plan was enabled in the Azure subscription.

![Defender Foundational Plan](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/defenders-plans.png)

* Initiated the onboarding process for the **AWS** account within the Defender for Cloud multi-cloud connector, configuring connector details and selecting posture management and workload protection (CWPP) plans.

![Add AWS Account](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-connector-setup.png)
![AWS Plans in MDC](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-plans-config.png)

### 2.2 Infrastructure as Code (IaC) Deployment with CloudFormation
* Generated the Azure-provided **CloudFormation** template to deploy the required IAM roles in the AWS account with read and audit privileges (`CspmMonitorAwsRole`).

![AWS CloudFormation Template](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-cloudformation-template.png)

* Executed and confirmed the successful stack creation in the AWS CloudFormation console (`CREATE_COMPLETE`).

![CloudFormation Events](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-cloudformation-events.png)

* Completed the connector creation, successfully linking the connected AWS account alongside the Azure subscription.

![AWS Connector Created](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-connector-success.png)
![Connected Clouds Summary - AWS](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/cloud-summary-aws.png)

### 2.3 Configuring CloudTrail and S3 for Log Ingestion
* Created an Amazon S3 bucket (`bucket-proyecto-soc-multicloud`) dedicated to storing and centralizing CloudTrail audit logs.

![AWS S3 Bucket](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-s3-bucket.png)

* Configured a multi-region trail in **AWS CloudTrail** to monitor API calls and account activity, routing events to the S3 bucket for subsequent ingestion into Sentinel.

![Multi-region CloudTrail](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/aws-cloudtrail-setup.png)

---

## ☁️ Step 3: Google Cloud Platform (GCP) Integration

### 3.1 Connecting the GCP Project to Defender for Cloud
* Initiated the GCP project onboarding in the multi-cloud connector specifying the appropriate identifiers.

![Add GCP Project](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/gcp-connector-setup.png)

* Executed the setup script in the **GCP Cloud Shell** to create the custom role (`MDCCspmCustomRole`), enable necessary APIs (`iam.googleapis.com`, `cloudresourcemanager.googleapis.com`, `compute.googleapis.com`, etc.), and configure the service account with Workload Identity Federation.

![GCP Cloud Shell Script](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/gcp-cloudshell-script.png)

* Verified proper synchronization and establishment of the GCP connector in Azure.

![GCP Connector Successful](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/gcp-connector-success.png)
![Multicloud Summary with GCP](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/cloud-summary-gcp.png)

### 3.2 Configuring Pub/Sub and Sinks for Audit Logs
* To ensure real-time forwarding of GCP audit logs (`cloudaudit.googleapis.com/activity`) to Azure, CLI commands were executed to create a Pub/Sub topic (`gcp-audit-logs-topic`), an associated subscription, and the corresponding Log Sink.

![GCP Pub/Sub and Sinks Setup](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/gcp-pubsub-cli.png)

---

## 🔍 Step 4: Configuring Connectors in Microsoft Sentinel
* From the Sentinel **Content Hub**, the **Amazon Web Services** solution was installed and the **Amazon Web Services S3** connector was configured using the required SQS/IAM permissions.

![Install AWS Solution in Sentinel](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-aws-solution.png)
![AWS S3 Connector in Sentinel](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-aws-s3-connector.png)

* Similarly, the **Google Cloud Platform Audit Logs** solution was installed, and the **GCP Pub/Sub Audit Logs** connector was configured linking the project ID, project number, subscription, and Workload Identity parameters.

![Install GCP Solution in Sentinel](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-gcp-solution.png)
![GCP Pub/Sub Connector Config](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-gcp-config.png)

* Validated correct data flow and ingestion by running verification KQL queries against the `GCPAuditLogs` and `AWSCloudTrail` tables.

![KQL GCPAuditLogs Query](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/kql-query-verification.png)

---

## ⚡ Step 5: Automation (SOAR) and Analytics Rules
To demonstrate incident response capabilities, a detection rule and an automated playbook were configured to handle the deletion of critical resources.

### 5.1 Creating the Playbook (Azure Logic Apps)
* Created a Logic App (`Playbook-Alerta-GCP-Email`) configured to trigger upon a Microsoft Sentinel incident and send an email notification.

![Create Logic App Playbook](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/logic-app-creation.png)
![Logic Apps Playbook Designer](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/logic-app-designer.png)

### 5.2 Creating a Scheduled Analytics Rule in Sentinel
* Created a new scheduled query rule designed to detect the deletion of virtual machines in GCP under the Impact tactic (MITRE ATT&CK).

![Create Sentinel Analytics Rule](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-rule-creation.png)
![GCP VM Rule Configuration](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-rule-config.png)
![Active Rules in Sentinel](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-rules-list.png)

* **Implemented KQL Query:**
```kusto
GCPAuditLogs
| where tostring(pack_all()) has "compute.googleapis.com"
  and (tostring(pack_all()) has "instances.delete" or tostring(pack_all()) has "delete")
| project TimeGenerated, SourceTable = "GCP", Details = tostring(pack_all())
```

![Rule KQL Query](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/kql-rule-query.png)

---

## 🧪 Step 6: Security Testing and Validation
* To test the analytics rule and automated response, a virtual machine instance (`instance-20260830-033842`) was created and subsequently deleted in the GCP environment.

![GCP VM Creation and Deletion](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/gcp-vm-test.png)

* As a result, Microsoft Sentinel detected the event via the scheduled rule, automatically generating the corresponding incident.

![Sentinel Incident Generated](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/sentinel-incident-generated.png)

* The incident triggered the configured Playbook, successfully sending an email alert to the security team (`victormgs44@gmail.com`).

![Received Email Notification](https://raw.githubusercontent.com/vmatos9/proyecto-soc-multicloud/main/images/email-alert-received.png)

---

## 👤 Author
* **Victor Matos** - *Cloud & Cybersecurity Professional*
