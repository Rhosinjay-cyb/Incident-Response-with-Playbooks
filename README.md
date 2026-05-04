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
This project marks the continuation of my project on managing security operation with tools from the Microsoft security stack. Prior to the commencement of this project, the following resources and services (Azure Bastion, Project-BST; Virtual Machine , Sales-VM; Virtual Network, Project-vnet; Subnet, Sales-vnet & AzureBastionSubnet; Microsoft Sentinel integrated-Log Analytics Workspace, Project-workspace; all in the same resource group, Project-RG) have already been deployed and will be used during the testing phase of this project.
For this project, we will paint a scenario to understand the workflow of this project. An IT company, Panda, has just reviewed its security policies and has placed a strict rule against connecting remotely to virtual machines hosting critical applications or data unless the connection is established with Azure Bastion for enhanced security. Additionally, the sales team has also instructed its members to restrain from connecting with unapproved IP addresses while using Azure Bastion to connect remotely to the team's VM as such IPs will regarded as untrusted IPs and the remote connection will be blocked. At the moment the team has only approved one IP address which is from the company's internet service provider. 


## Steps Taken



## Conclusion



## Future Works




## Past Project

https://rhosinjay-cyb.github.io/Azure-Firewall/
https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

