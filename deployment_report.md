# Azure Firewall Implementation and Security Validation Report

## Deployment Overview

### Environment Details

The Azure Firewall security environment was implemented within an Azure for Students subscription hosted in the West Europe region. All resources were provisioned inside a dedicated resource group using Azure CLI version 2.86.0. The project focused on deploying Azure Firewall, enforcing traffic control policies, implementing network security measures, and validating firewall functionality through a series of controlled tests.

| Configuration Item    | Specification         |
| --------------------- | --------------------- |
| Subscription Type     | Azure for Students    |
| Azure Region          | West Europe           |
| Resource Group        | rg-azure-firewall-lab |
| Implementation Period | June 2026             |
| Azure CLI Release     | 2.86.0                |

---

# Section 1 – Virtual Network Design and Configuration

## Purpose

The objective of this phase was to establish the networking foundation required for Azure Firewall deployment by creating a virtual network architecture containing dedicated subnets for firewall services and workload resources.

## Deployment Activities

A resource group was first provisioned within the West Europe region. A virtual network named **vnet-firewall-lab** was subsequently created with an address space of **10.0.0.0/16**. Two subnets were then configured to support firewall operations and application workloads.

The first subnet, **AzureFirewallSubnet (10.0.1.0/26)**, was reserved exclusively for Azure Firewall deployment. The second subnet, **WorkloadSubnet (10.0.2.0/24)**, was created to host virtual machines and application resources whose traffic would be inspected by the firewall.

## Outcome

The network infrastructure was successfully established and met all Azure Firewall deployment prerequisites.

### Achievements

* Successfully provisioned the resource group.
* Created the virtual network using a private address space of 10.0.0.0/16.
* Configured a dedicated firewall subnet in accordance with Azure requirements.
* Established a separate workload subnet for application hosting.

**Important Consideration:** Azure Firewall can only be deployed into a subnet named **AzureFirewallSubnet**. Any deviation from this naming convention prevents successful deployment.

---

# Section 2 – Azure Firewall Provisioning

## Purpose

This stage focused on deploying Azure Firewall and assigning the networking resources required for secure traffic inspection and management.

## Deployment Activities

A Standard SKU static public IP address was created to provide external connectivity for the firewall. A firewall policy was then configured with Microsoft Threat Intelligence operating in Alert mode. Finally, an Azure Firewall instance was deployed and associated with both the virtual network and public IP configuration.

## Outcome

The firewall deployment was completed successfully, enabling centralized inspection and control of network traffic.

### Achievements

* Provisioned a Standard SKU static public IP address.
* Created a firewall policy with threat intelligence monitoring enabled.
* Deployed Azure Firewall using the Standard tier.
* Assigned a private IP address within the firewall subnet.
* Associated public and private interfaces with the firewall instance.

---

# Section 3 – Traffic Routing and User Defined Routes

## Purpose

To ensure that all outbound traffic originating from workload resources would be processed and inspected by Azure Firewall before reaching external destinations.

## Deployment Activities

A route table was created and configured with a default route directing all internet-bound traffic to the firewall's private IP address. The route table was then linked to the workload subnet, enforcing firewall inspection for outbound communications.

## Outcome

Traffic routing was successfully redirected through Azure Firewall, enabling centralized security enforcement and monitoring.

### Achievements

* Created a dedicated route table.
* Configured a default route for all outbound traffic.
* Specified Azure Firewall as the next-hop appliance.
* Linked the route table to the workload subnet.
* Disabled BGP route propagation to prevent route conflicts.

**Key Observation:** Using Azure Firewall as the designated next hop guarantees that outbound traffic cannot bypass inspection and filtering mechanisms.

---

# Section 4 – Network-Level Traffic Control

## Purpose

To regulate network communications based on source addresses, destination addresses, protocols, and port numbers.

## Deployment Activities

A network rule collection group was established within the firewall policy. Rules were configured to permit DNS requests to approved DNS servers while restricting all unauthorized traffic.

## Outcome

Network traffic filtering was successfully implemented according to the defined security requirements.

### Implemented Policies

| Policy Name        | Protocol | Source Network | Destination          | Port | Action |
| ------------------ | -------- | -------------- | -------------------- | ---- | ------ |
| DNS Access Rule    | UDP      | 10.0.2.0/24    | Approved DNS Servers | 53   | Allow  |
| ICMP Rule          | ICMP     | 10.0.2.0/24    | Any                  | Any  | Allow  |
| Default Block Rule | Any      | Any            | Any                  | Any  | Deny   |

---

# Section 5 – Application-Aware Security Policies

## Purpose

To restrict outbound web access and permit communication only with approved domains and cloud services.

## Deployment Activities

An application rule collection group was created to manage web-based traffic. Rules were configured to allow HTTPS access to Microsoft, GitHub, and Azure services while denying access to unauthorized websites.

## Outcome

Application-layer filtering was successfully enforced, limiting internet access to approved destinations.

### Approved Destinations

| Policy                  | Protocol   | Authorized Domains | Action |
| ----------------------- | ---------- | ------------------ | ------ |
| Microsoft Access        | HTTPS      | *.microsoft.com    | Allow  |
| GitHub Access           | HTTPS      | *.github.com       | Allow  |
| Azure Service Access    | HTTPS      | *.azure.com        | Allow  |
| Default Web Restriction | HTTP/HTTPS | All Others         | Deny   |

---

# Section 6 – Destination Network Address Translation (DNAT)

## Purpose

To provide controlled inbound access to internal resources while maintaining network isolation.

## Deployment Activities

A NAT rule collection group was configured to translate requests received on the firewall's public IP address to an internal workload server hosted within the private subnet.

## Outcome

External users were able to access approved internal services through secure destination NAT translation.

### Configured Translation Rule

| External Endpoint | Internal Endpoint | Protocol |
| ----------------- | ----------------- | -------- |
| Public IP:8080    | 10.0.2.4:80       | TCP      |

---

# Section 7 – Threat Intelligence Protection

## Purpose

To enhance security posture by leveraging Microsoft's global threat intelligence feeds to identify and block malicious traffic.

## Deployment Activities

The firewall policy was updated from Alert mode to Deny mode, enabling automatic blocking of traffic associated with known malicious IP addresses and domains.

## Outcome

Threat intelligence enforcement was successfully activated for both inbound and outbound communications.

### Security Configuration

| Setting                  | Value                         |
| ------------------------ | ----------------------------- |
| Threat Intelligence Mode | Alert and Deny                |
| Intelligence Source      | Microsoft Threat Intelligence |
| Coverage                 | Inbound and Outbound Traffic  |
| Enforcement Action       | Detect, Log, and Block        |

---

# Section 8 – Functional Testing and Security Verification

## Purpose

To verify that all firewall configurations, routing policies, and security controls functioned as expected.

## Validation Approach

A series of controlled connectivity tests were conducted from workload virtual machines and external endpoints. The tests examined DNS access, application rule enforcement, threat intelligence protection, and NAT functionality.

## Outcome

All validation scenarios produced the expected results, confirming that the firewall was operating according to the defined security requirements.

### Summary of Results

* Approved DNS traffic was successfully permitted.
* Access to authorized Microsoft and GitHub services was successful.
* Unauthorized websites were correctly blocked.
* Non-approved DNS services were denied.
* Threat intelligence protection successfully prevented malicious communications.
* DNAT translation functioned as expected for inbound traffic.

The successful completion of all test cases demonstrates effective implementation of a least-privilege network security model.

---

# Section 9 – Monitoring, Logging, and Visibility

## Purpose

To establish centralized monitoring and logging capabilities for firewall activity and security events.

## Deployment Activities

A Log Analytics Workspace was created and integrated with Azure Firewall diagnostic settings. Network rule logs, application rule logs, and performance metrics were enabled to support operational monitoring and security investigations.

## Outcome

Comprehensive visibility into firewall operations was achieved, allowing administrators to monitor traffic patterns, investigate security events, and generate analytical reports.

### Monitoring Capabilities

* Collection of network rule activity logs.
* Collection of application rule activity logs.
* Firewall performance metrics monitoring.
* Traffic analysis through Kusto Query Language (KQL).
* Security event investigation and reporting.

## Conclusion

The Azure Firewall implementation was successfully completed and validated. Core security capabilities including traffic inspection, routing enforcement, application filtering, NAT translation, threat intelligence protection, and centralized monitoring were configured and tested successfully. The deployment demonstrates a practical implementation of Azure network security best practices and provides a secure foundation for protecting cloud-hosted workloads.
