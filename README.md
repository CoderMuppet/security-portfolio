# Security Portfolio: Costea Cristiam

Documented cloud security investigations, built in a live Azure tenant
(Mad Hat Labs, a multi-user training environment).

Target role: SOC Analyst / Security Analyst
Currently: [your role] | [City / Remote]
Contact: [email] · [LinkedIn URL]

## Investigations
| # | Title | Focus | Write-up |
|---|-------|-------|----------|
| 1 | Operation Dead Deploy | Governance forensics, deployment audit trail
| 2 | The Stolen Identity | App registration attack kill chain (Entra ID) | coming, week 2 |
| 3 | Privilege Audit | RBAC and least privilege | coming, week 3 |
| 4 | Spin Up and Lock Down | Compute attack surface | coming, week 4 |
| 5 | Network the Operative | Network segmentation | coming, week 5 |
| 6 | Bucket Looting | Storage exposure hunting | coming, week 6 |
| 7 | Find the Anomaly | Log analysis and KQL | coming, week 7 |
| 8 | Hunt the Threat | SIEM operations (Sentinel) | coming, week 8 |
| 9 | Score the Tenant | Cloud security posture | coming, week 9 |
| 10 | The Breach (capstone) | Full incident investigation | coming, week 10 |

# Operation Dead Deploy: Investigating Unauthorized Azure Resource Provisioning

**Category:** Cloud Security | Azure Governance | Incident Investigation  
**Environment:** Live multi-user Azure training tenant  
**Role:** On-call engineer (simulated) | Reader access

## Scenario

Following a weekend deployment by an intern with temporary Contributor access, an unexpected test environment was discovered in an Azure subscription. The resources appeared to violate organizational naming standards despite an active Azure Policy assignment.
My task was to investigate what had been deployed, reconstruct the available deployment history, and identify why the existing governance controls had not prevented the violation.

## Environment
- **Platform:** Microsoft Azure
- **Environment:** Live multi-user Azure training tenant (Mad Hat Labs)
- **Services:** Azure Resource Groups, Azure Resource Manager (ARM), Azure Policy
- **Tools:** Azure Portal
- **Access level:** Reader (read-only)
- **Investigation focus:** Resource discovery, metadata analysis, deployment tracing, and policy compliance

## Investigation
### 1. Identifying the suspicious resource group
I began by reviewing the resource groups within the Azure subscription and comparing their names against the established naming convention.
The majority followed a predictable structure containing identifiers for the workload, environment, and region.
One resource group stood out because it lacked these expected components, making it inconsistent with the rest of the subscription.
**Finding:** An unusually named resource group was identified as a potential example of provisioning outside established governance standards.
**Reasoning:** Consistent naming conventions support resource identification, ownership tracking, and operational management. An unexplained deviation can indicate poor configuration management or unauthorized provisioning.

### 2. Inspecting the deployed resource and its metadata
After identifying the suspicious resource group, I opened it to inspect its contents.
The resource group contained a single deployed resource.
I navigated to the resource's Tags section to examine the metadata associated with the deployment.
The tags contained information linking the resource to an internal test deployment.
**Finding:** The resource contained deployment-related metadata, providing additional context about its intended purpose.
**Reasoning:** Resource tags can help investigators establish ownership, purpose, cost allocation, and expected lifecycle. They are useful investigative clues, although they should not be treated as definitive proof of who performed a deployment.

### 3. Reconstructing the deployment history
The next step was to determine how the resource had been provisioned.
I returned to the resource group and opened the **Deployments** section, which contains records of Azure Resource Manager deployments.
I examined the deployment information, focusing on its name, timestamp, status, and available configuration details.
The deployment record provided clues about the origin and timing of the resource creation.
**Finding:** The deployment history helped establish how the resource had been provisioned and provided information that could contribute to an incident timeline.
**Reasoning:** Deployment records are important forensic artifacts. Combined with Azure Activity Logs and identity information, they can help investigators establish when changes occurred and which identity initiated them.
A deployment name alone is not sufficient to prove who performed the action.

### 4. Investigating the Azure Policy assignment
After tracing the deployment, I investigated why the resource had been allowed to exist despite violating the naming convention.
I navigated to the resource group's Policies section and reviewed the compliance information.
The naming convention policy was present and reported a non-compliant state.
This confirmed that Azure Policy could detect the violation, but it did not explain why the deployment had succeeded.

### 5. Identifying the governance failure
To investigate further, I opened the policy assignment and reviewed its configured parameters.
The critical finding was that the policy's **Effect** parameter was set to `Audit`.
This configuration allows resources that violate the evaluated policy rules to be deployed while recording the violations for compliance monitoring.
A `Deny` effect, in contrast, is designed to reject requests that violate the policy rules.
**Root cause:** The naming policy was configured to detect violations rather than prevent them.
The issue was not that the policy was missing. The policy existed, but its configured enforcement behavior did not match the expectation that naming violations would be blocked.


## What Broke / What Surprised Me

The most important discovery was that an active Azure Policy does not necessarily prevent a non-compliant resource from being deployed.
Initially, the presence of a naming policy might suggest that naming standards were being enforced. However, inspecting the assignment parameters revealed that compliance monitoring and preventive enforcement are different behaviors.
A resource can therefore exist in a non-compliant state even when the relevant Azure Policy is correctly assigned and functioning as configured.


## Findings and Recommendations
### Findings
The investigation established that:

1. A resource group existed outside the expected Azure naming convention.
2. The resource group's contents and metadata were consistent with an internal test deployment.
3. Azure Resource Manager deployment records provided evidence useful for reconstructing the provisioning event.
4. The naming convention policy detected the violation but was configured with the `Audit` effect rather than `Deny`.

### Recommendations
**1. Review policy enforcement settings**
Determine whether the naming policy is intended to provide monitoring or preventive enforcement.
If naming standards must be mandatory, evaluate switching the policy effect from `Audit` to `Deny`, following appropriate testing and change-control procedures.
If the audit-only configuration is intentional, document that decision and establish a process for reviewing non-compliant resources.

**2. Strengthen temporary privileged access controls**
Review how temporary Contributor permissions are granted and revoked.
Use least-privilege access, time-limited permissions, and approval procedures where appropriate. Regularly review access assignments to reduce unnecessary provisioning privileges.

**3. Enforce resource metadata requirements**
Introduce policies requiring appropriate ownership, environment, and lifecycle tags when resources are deployed.
Required metadata improves accountability and simplifies future compliance reviews and incident investigations.

## What I Learned

- **Azure Policy effects matter:** `Audit` detects non-compliance, while `Deny` can prevent non-compliant deployment requests.
- **Deployment records support investigations:** ARM deployment history provides useful information about provisioning activity and can help reconstruct an incident timeline.
- **Resource tags provide investigative context:** Metadata can reveal a resource's intended purpose, although it is not definitive evidence of identity.
- **Governance is more than creating policies:** A control must be correctly configured, scoped, monitored, and maintained to achieve its intended security outcome.
- **What I would do differently:** In a future investigation, I would examine policy assignments and their enforcement effects earlier, then correlate deployment records with Azure Activity Logs to verify the responsible identity and timeline.

## Conclusion
This investigation demonstrated how a seemingly minor naming anomaly can lead to the discovery of a broader cloud governance weakness.
By reviewing resource metadata, deployment records, and Azure Policy configuration, I identified why a non-compliant resource had been permitted in the subscription.
The central lesson was that **detecting a violation is not the same as preventing it**. Effective cloud governance requires appropriate enforcement settings, access controls, and regular compliance reviews.

