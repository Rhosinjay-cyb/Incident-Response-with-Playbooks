## Project Title

Automating incident response with playbooks

## Objective

The aim of this project is to design a playbook with Azure Logic Apps and integrate it with Microsoft Sentinel via automation rule to  facilitate automated response to incidents on Microsoft Sentinel, thereby supporting the effective management of security operations (SecOps).

## Tools Used

Microsoft Sentinel, Microsoft Defender Portal, Azure Logic Apps


## Lab Setup
* Creation of analytics rule
* Designing the workflow of the playbook with Azure Logic Apps
* Creation of automation rule (Integration of playbook with incident)
* Testing of the playbook



## Background Information
This project marks the continuation of my project on managing security operation with tools from the Microsoft security stack. Prior to the commencement of this project, the following resources and services (Azure Bastion, Project-BST; Virtual Machine , Sales-VM; Virtual Network, Project-vnet; Subnet, Sales-vnet & AzureBastionSubnet; Microsoft Sentinel integrated-Log Analytics Workspace, Project-workspace; all in the same resource group, Project-RG) have already been deployed and will be used during the testing phase of this project. Additionally, the diagnostic settings of Azure Bastion (Project-BST) has already been enabled and its events logs are available in the Microsoft Sentinel integrated-Log Analytics Workspace (Project-workspace).

Putting this project into context, the sales team of a company has a VM (Sales-VM) which host critical applications and data, and would want to enhance its security by preventing unauthorised access. The team has approved Azure Bastion and one particular IP address for remote connection to the VM. The team wants every other IPs to be deemed untrusted and such connections should be blocked. Hence, the security solution for the sales team is the core of this project. Initially, each of the IPs that are used for the remote connection could be monitored from the logs collected from Project-BST which are stored in MicrosoftAzureBastionAuditLogs table in the Project-workspace.

Afterwards, an analytics rule will be created to detect untrusted IPs, and the automation rule attached to the analytics rule will be utilized to trigger the playbook which will block the remote connection. The other action of the automation role is to assign the incident to a member of the security team while the last action of the playbook is to send an email notification about the result of the playbook to relevant members of the security team.

## Steps Taken



## Conclusion



## Future Works




## Past Project

https://rhosinjay-cyb.github.io/Azure-Firewall/
https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

