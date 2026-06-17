# Azure Firewall Security Implementation Project

## Project Summary

This project showcases the deployment and configuration of a centralized network security solution within Microsoft Azure using Azure Firewall. The implementation demonstrates how organizations can move beyond traditional Network Security Groups (NSGs) and adopt a unified firewall platform that provides traffic inspection, policy enforcement, threat protection, and monitoring capabilities across cloud environments.

Azure Firewall was configured to manage both inbound and outbound traffic, enforce security policies, perform network address translation (NAT), and integrate with Azure monitoring services for visibility and auditing.

---

# Solution Architecture Overview

The environment consists of a dedicated virtual network containing separate subnets for Azure Firewall and application workloads. All outbound traffic from workload resources is redirected through the firewall for inspection, while inbound traffic is controlled through destination NAT rules.

```text
Internet
    │
    ▼
Public Endpoint (pip-firewall-lab)
    │
    ▼
Azure Firewall (fw-lab)
    │
    ▼
Virtual Network: vnet-firewall-lab
    │
    ├── Firewall Subnet (10.0.1.0/26)
    │
    └── Workload Subnet (10.0.2.0/24)
            │
            ▼
      Virtual Machine (vm-workload)
```

A User Defined Route (UDR) ensures that all outbound communications originating from the workload subnet are processed by Azure Firewall before reaching external destinations.

---

# Repository Contents

The repository contains deployment documentation, configuration files, validation logs, and supporting resources required to reproduce the environment.

```text
azure-firewall-assignment/
├── README.md
├── architecture_diagram.md
├── deployment_report.md
├── troubleshooting_report.md
├── configs/
├── logs/
└── screenshots/
```

### Documentation Components

| File                      | Purpose                                           |
| ------------------------- | ------------------------------------------------- |
| README.md                 | Project overview and implementation summary       |
| architecture_diagram.md   | Network design and topology documentation         |
| deployment_report.md      | Detailed deployment procedures and results        |
| troubleshooting_report.md | Resolution of deployment and configuration issues |
| configs                   | Exported firewall and routing configurations      |
| logs                      | Traffic validation and monitoring evidence        |
| screenshots               | Visual evidence supporting project requirements   |

---

# Project Completion Status

The following implementation activities were successfully completed as part of the Azure Firewall deployment.

| Activity                               | Completion Status |
| -------------------------------------- | ----------------- |
| Virtual Network Configuration          | Completed         |
| Azure Firewall Deployment              | Completed         |
| Route Management and UDR Configuration | Completed         |
| Network Rule Configuration             | Completed         |
| Application Rule Implementation        | Completed         |
| NAT Rule Configuration                 | Completed         |
| Threat Intelligence Enablement         | Completed         |
| Security Validation and Testing        | Completed         |
| Monitoring and Logging Setup           | Completed         |

---

# Security Policy Configuration

## Network-Level Access Controls

The firewall was configured with network-based rules to regulate protocol and port-level communications.

### Implemented Controls

* Permitted DNS requests from workload resources to approved DNS servers.
* Allowed ICMP traffic for diagnostic and troubleshooting purposes.
* Denied unauthorized network communications through an implicit default-deny policy.

---

## Application-Level Filtering

Application rules were implemented to control web traffic based on fully qualified domain names (FQDNs).

### Approved Destinations

* Microsoft cloud services (`*.microsoft.com`)
* GitHub services (`*.github.com`)
* Selected Azure service endpoints

All other HTTP and HTTPS traffic is blocked unless explicitly authorized.

---

## Destination NAT Configuration

Inbound traffic is tightly controlled through Destination Network Address Translation (DNAT).

### Translation Policy

| External Endpoint   | Internal Destination |
| ------------------- | -------------------- |
| Public IP Port 8080 | VM Port 80           |

This configuration enables secure external access to internal services without exposing the virtual machine directly to the internet.

---

## Threat Protection Configuration

Azure Firewall Threat Intelligence was enabled in **Alert and Deny** mode.

### Security Functions

* Detection of connections to known malicious IP addresses.
* Blocking of traffic associated with threat intelligence indicators.
* Logging and reporting of security-related events.
* Continuous protection using Microsoft's global threat intelligence feeds.

---

# Azure Resources Provisioned

The following cloud resources were deployed to support the solution.

| Resource Type           | Resource Name         | Region      |
| ----------------------- | --------------------- | ----------- |
| Resource Group          | rg-azure-firewall-lab | West Europe |
| Virtual Network         | vnet-firewall-lab     | West Europe |
| Azure Firewall          | fw-lab                | West Europe |
| Firewall Policy         | fwpolicy-lab          | West Europe |
| Public IP Address       | pip-firewall-lab      | West Europe |
| Route Table             | rt-firewall-lab       | West Europe |
| Log Analytics Workspace | law-firewall-lab      | West Europe |

---

# Deployment Replication Guide

To recreate this environment, refer to the deployment documentation contained in **deployment_report.md**. The guide provides detailed Azure CLI commands, configuration steps, validation procedures, and verification results required to deploy the solution from scratch.

---

# Visual Evidence Requirements

To support project assessment and verification, screenshots should be captured from the Azure Portal and testing environment.

### Required Evidence

1. Network Security Rule Configuration
2. Application Rule Collections
3. NAT Rule Configuration
4. Route Table and Next-Hop Settings
5. Traffic Verification and Connectivity Testing

Each screenshot should clearly demonstrate the successful implementation of the associated firewall feature.

---

# Firewall Validation and Verification

Comprehensive testing was performed to confirm correct firewall operation.

### Validation Areas

* DNS resolution through approved servers
* Access to permitted web destinations
* Blocking of unauthorized internet traffic
* Enforcement of threat intelligence policies
* NAT translation functionality
* Route table effectiveness
* Traffic inspection and logging

Testing results confirmed that all configured security controls behaved as expected and aligned with a least-privilege security model.

---

# Security Architecture Principles Applied

The deployment was designed according to established cloud security best practices.

### Least Privilege Access

Only explicitly approved traffic is allowed. All other communications are denied by default.

### Centralized Traffic Inspection

User Defined Routes ensure that workload traffic cannot bypass firewall inspection.

### Domain-Based Access Control

Application rules leverage FQDN filtering to provide granular control over web access.

### Threat Intelligence Integration

Known malicious destinations are automatically blocked using Microsoft's continuously updated threat intelligence database.

### Controlled External Access

Internal services remain private and are only exposed through carefully defined DNAT rules.

### DNS Security

DNS communications are restricted to approved resolvers, reducing the risk of unauthorized name resolution and data exfiltration techniques.

---

# Project Information

| Property            | Value                     |
| ------------------- | ------------------------- |
| GitHub Repository   | azure-firewall-assignment |
| Subscription Type   | Azure for Students        |
| Deployment Period   | June 2026                 |
| Firewall Public IP  | 20.61.208.52              |
| Firewall Private IP | 10.0.1.4                  |

## Conclusion

The Azure Firewall implementation successfully demonstrates centralized cloud network security through traffic filtering, route enforcement, application control, threat protection, monitoring, and controlled inbound access. The completed deployment provides a secure, scalable, and manageable foundation for protecting Azure-hosted workloads while following industry-standard security practices.
