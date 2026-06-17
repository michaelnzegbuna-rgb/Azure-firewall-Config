# Azure Firewall Deployment Challenges and Resolution Report

## Introduction

This document outlines the technical issues encountered during the implementation of the Azure Firewall laboratory environment, along with the investigative procedures and corrective actions applied to resolve them. The report highlights practical troubleshooting techniques used to identify configuration errors, restore functionality, and ensure successful deployment of Azure Firewall services.

---

# Challenge 1 – Insufficient Azure Firewall Subnet Capacity

## Description of the Issue

During the network design phase, the Azure Firewall subnet was initially allocated using a **/28 CIDR range**. When deployment was attempted, Azure returned an error indicating that the subnet size did not meet the minimum requirement for Azure Firewall.

**Error Message**

```text
The subnet AzureFirewallSubnet size /28 is too small.
Azure Firewall requires a minimum subnet size of /26.
```

## Investigation Findings

Azure Firewall is designed as a scalable and highly available managed service. To support infrastructure operations, service updates, and future scaling activities, Azure reserves multiple IP addresses within the subnet. As a result, smaller subnet allocations cannot accommodate firewall requirements.

## Corrective Action

The subnet address range was expanded from **/28** to **/26**, satisfying Azure Firewall deployment prerequisites.

```bash
az network vnet subnet update \
  --resource-group rg-azure-firewall-lab \
  --vnet-name vnet-firewall-lab \
  --name AzureFirewallSubnet \
  --address-prefixes 10.0.1.0/26
```

## Key Takeaway

Azure Firewall deployments should always use an **AzureFirewallSubnet** with a minimum address space of **/26** to avoid deployment failures and future scaling limitations.

---

# Challenge 2 – Unsupported Public IP Configuration

## Description of the Issue

A Public IP resource was initially created using the **Basic SKU** option. While the IP resource was successfully provisioned, Azure Firewall failed when attempting to associate with it.

**Error Message**

```text
Azure Firewall requires a Standard SKU public IP address.
Basic SKU public IP addresses are not supported.
```

## Investigation Findings

Azure Firewall requires enhanced availability, resiliency, and networking capabilities that are only available through the Standard SKU Public IP offering.

## Corrective Action

The existing Public IP was recreated using the Standard SKU configuration and static allocation.

```bash
az network public-ip create \
  --resource-group rg-azure-firewall-lab \
  --name pip-firewall-lab \
  --sku Standard \
  --allocation-method Static
```

## Key Takeaway

Always deploy Azure Firewall with a **Standard SKU Static Public IP**, as Basic SKU addresses are not compatible with firewall services.

---

# Challenge 3 – Firewall Policies Not Affecting Traffic

## Description of the Issue

Following firewall deployment and policy configuration, outbound internet traffic from workload virtual machines remained unrestricted. Security rules appeared to have no effect on network communications.

## Investigation Process

The effective route table assigned to the virtual machine network interface was reviewed.

```bash
az network nic show-effective-route-table \
  --resource-group rg-azure-firewall-lab \
  --name vm-workload-nic \
  --output table
```

The output revealed that traffic continued to follow Azure's default system routes instead of being redirected through Azure Firewall.

## Investigation Findings

Although a User Defined Route (UDR) had been created, it had not been attached to the workload subnet. Consequently, workload resources continued using Azure's default routing configuration.

## Corrective Action

The route table was explicitly linked to the workload subnet.

```bash
az network vnet subnet update \
  --resource-group rg-azure-firewall-lab \
  --vnet-name vnet-firewall-lab \
  --name WorkloadSubnet \
  --route-table rt-firewall-lab
```

Verification confirmed that all outbound traffic was subsequently directed to the firewall's private IP address.

## Key Takeaway

Creating a route table alone does not enforce routing policies. The route table must also be associated with the target subnet for traffic inspection to occur.

---

# Challenge 4 – Domain Name Resolution Failures

## Description of the Issue

After implementing forced tunnelling through Azure Firewall, workload virtual machines could no longer resolve domain names.

**Observed Symptoms**

```text
curl: (6) Could not resolve host: microsoft.com
nslookup: connection timed out; no servers could be reached
```

## Investigation Process

The DNS configuration on the virtual machine was reviewed.

```bash
cat /etc/resolv.conf
```

The VM was configured to use Azure's default DNS service.

## Investigation Findings

DNS requests were being blocked because no firewall rule existed to permit UDP traffic on port 53. Since the firewall was operating with a default-deny approach, DNS queries were rejected.

## Corrective Action

A dedicated network rule was created to permit DNS traffic to approved DNS servers.

```bash
az network firewall policy rule-collection-group collection add-filter-collection \
  --resource-group rg-azure-firewall-lab \
  --policy-name fwpolicy-lab \
  --rule-collection-group-name NetworkRuleCollectionGroup \
  --name AllowNetworkRules \
  --collection-priority 100 \
  --action Allow \
  --rule-name AllowDNS \
  --rule-type NetworkRule \
  --source-addresses "10.0.2.0/24" \
  --destination-addresses "8.8.8.8" "8.8.4.4" \
  --ip-protocols UDP \
  --destination-ports 53
```

## Key Takeaway

DNS traffic should be permitted early in firewall configuration, as most application and internet services depend on successful name resolution.

---

# Challenge 5 – Application Rules Failing to Match Traffic

## Description of the Issue

Despite creating application rules to allow traffic to approved domains such as Microsoft services, HTTPS requests continued to fail.

## Investigation Findings

Azure Firewall application rules depend on Fully Qualified Domain Name (FQDN) resolution. Because DNS traffic was blocked, domain names could not be resolved and application rule processing could not occur.

## Corrective Action

Restoring DNS functionality immediately resolved the application rule issue, allowing FQDN-based filtering to function correctly.

## Key Takeaway

Application rules rely on DNS resolution. Without DNS access, Azure Firewall cannot evaluate FQDN-based policies.

### Rule Processing Sequence

1. NAT Rules
2. Network Rules
3. Application Rules

Proper DNS functionality is therefore a prerequisite for application-layer filtering.

---

# Challenge 6 – Missing Threat Intelligence Events

## Description of the Issue

Threat Intelligence mode was enabled; however, no threat-related events appeared in monitoring logs.

## Investigation Findings

Two primary factors contributed to the issue:

1. Diagnostic logging had not yet been configured.
2. Test traffic was not targeting any known malicious destinations.

Without log collection and relevant traffic, no threat intelligence events could be generated.

## Corrective Action

### Configure Diagnostic Logging

```bash
az monitor diagnostic-settings create \
  --name fw-diagnostics \
  --resource <firewall-resource-id> \
  --workspace <log-analytics-workspace-id> \
  --logs '[{"category":"AzureFirewallNetworkRule","enabled":true},{"category":"AzureFirewallApplicationRule","enabled":true}]'
```

### Enable Active Blocking

```bash
az network firewall policy update \
  --resource-group rg-azure-firewall-lab \
  --name fwpolicy-lab \
  --threat-intel-mode "Deny"
```

### Validate Through Log Analytics

```kql
AzureDiagnostics
| where Category == "AzureFirewallNetworkRule"
| where msg_s contains "ThreatIntel"
| project TimeGenerated, msg_s, Action_s
```

## Key Takeaway

Threat Intelligence monitoring requires both diagnostic logging and traffic that matches Microsoft's threat intelligence feeds before events become visible.

---

# Summary of Resolved Deployment Issues

| Reference | Issue Encountered                  | Underlying Cause                | Resolution Implemented             |
| --------- | ---------------------------------- | ------------------------------- | ---------------------------------- |
| 1         | Firewall subnet deployment failure | Insufficient subnet size        | Expanded subnet to /26             |
| 2         | Public IP compatibility issue      | Incorrect SKU selection         | Recreated using Standard SKU       |
| 3         | Traffic bypassing firewall         | Missing route table association | Linked route table to subnet       |
| 4         | DNS lookup failures                | DNS traffic blocked             | Added DNS network rule             |
| 5         | Application rules not functioning  | Dependency on DNS resolution    | Restored DNS access                |
| 6         | No Threat Intelligence events      | Logging not configured          | Enabled diagnostics and monitoring |

---

# Useful Diagnostic Commands

The following commands proved valuable throughout the troubleshooting process and can be used during future Azure Firewall investigations.

### Verify Firewall Status

```bash
az network firewall show \
  --resource-group rg-azure-firewall-lab \
  --name fw-lab \
  --query "provisioningState"
```

### Review Effective Routes

```bash
az network nic show-effective-route-table \
  --resource-group rg-azure-firewall-lab \
  --name <nic-name> \
  --output table
```

### Inspect Firewall Policies

```bash
az network firewall policy rule-collection-group list \
  --resource-group rg-azure-firewall-lab \
  --policy-name fwpolicy-lab \
  --output table
```

### Validate Diagnostic Configuration

```bash
az monitor diagnostic-settings list \
  --resource <firewall-resource-id>
```

## Conclusion

The troubleshooting activities carried out during the Azure Firewall deployment successfully resolved configuration, routing, DNS, monitoring, and security policy issues. Through systematic investigation and corrective actions, the environment was brought to a fully operational state with traffic inspection, application filtering, threat intelligence protection, and monitoring capabilities functioning as intended. The lessons learned from these incidents provide valuable guidance for future Azure Firewall implementations and operational support activities.
