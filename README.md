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

To put this project into context, the sales team of a company has a VM (Sales-VM) which host critical applications and data, and would want to enhance its security by preventing unauthorised access. The team has approved Azure Bastion and one particular IP address for remote connection to the VM. The team wants every other IPs to be deemed untrusted and such connections should be blocked. Hence, the security solution for the sales team is the core of this project. 

Initially, each of the IPs that are used for the remote connection could be monitored from the logs collected from Project-BST which are stored in MicrosoftAzureBastionAuditLogs table in the Project-workspace.
Afterwards, an analytics rule will be created to detect untrusted IPs, and the automation rule attached to the analytics rule will be utilized to trigger the playbook. The main actions of the playbook is to block the inbound RDP port of the VM once the attack is detected. However, the blocked RDP port will only prevent future remote connection to the VM whilethe attacker stillremains connected. The next action is the identification of the malicious remote connections among all open RDP session and shutting it off. The last action of the playbook is the sending of an email notification about the result of the playbook to relevant members of the security operations team, while the other actions of the automation role is to assign the incident to a member of the security operations team for necessary actions.

The creation of the analytics rule and the design of the workflow in the playbook will be discussed in the next section, while the testing of the playbook and conclusion follows respectively.

## Steps Taken

The project commenced with the creation of an analytics rule (scheduled) to detect remote connections to the Sales-VM via Azure Bastion with an untrusted IP.


Followed with the designing of the workflow of the playbook from trigger to the last action.


The trigger for the workflow is Microsoft Sentinel Incident because the playbook is designed to run when an incident is created in Microsoft Sentinel.


To collect the entities in the incident, these variables of array data type are initialized to collect and store the entities. 
Note: Entity mapping is the responsibility of the security team while creating the analytics rule, mapping the right entity to the analytics rule helps in enriching incident investigation.


The for each loop has two seperate conditions which is used to collect the entities. The first conditons appends IPs to the array IPList while the under condition appends Username to the array UserList.


The above actions ONLY implements the collection of entities from the incident.

The next action is used to create a network securtiy group (NSG) rule to block all inbound connection to the Sales-VM including incoming RDP traffic. The Azure Resource Manager connector is used for this action and 'Create or update resource' was choosing among the available options. The parameters were filled accordingly, resource explorer would be quite helpful in getting the short resource id. Location and properties were selected among the advanced parameters and filled as well. The values in the properties field will now be used to create a NSG rule by updating the Sales-VM-nsg which will block inbound connections to the Sales-VM.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a network contributor role at the scope of the the resource, virtual machine (Sales-VM).


The next action in the workflow is the closing of remote session on the virtual machine when a user connects to the VM with an untrusted IP. This action also ensures that only the malicious session is killed while other sessions are skipped preventing the disruption of production operation. The action is implemented with the HTTP connector, which sends a web request to the API of Azure Resource Manager (ARM), the ARM validatesthe request and allows the runCommand service to utilize the Azure VM agent to exexute the powershell script in the body of the HTTP request, basically to kill open malicious remote session.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a Virtual Machine contributor role at the scope of the virtual network (Project-vnet).

The last of the actions on the workflow is to send an email notification to relevant member of the security operations team for review and other neccessary actions. Aside alerting the team, the email also provide a summary of the attack at a glance with details that includes the entities extracted from the incident details.


## Testing of the Playbook

For the effective testing of the playbook, a new user (Random-user) is created on the Sales-VM and it was added to the remote desktop users group to allow the user to able to logon remotely. 

Afterwards, the user account created alongside with the VM (Sales-user) was used to logon remotely to the VM via Azure Bastion while using the trusted IP. To simulate an attack scenario, the Random-user account was used to logon to the Sales-VM remotely via Azure Bastion as well but with an untrusted IP. 



The detection of the usage of an untrusted IP for connection to the VM led to the firing of a Microsoft Sentinel alert and an incident thereafter. 



While reviewing the incident tab, it was observed that the automation rule had assigned the incident to a member in the SecOps team. Also, the review of the incident page showed that the playbook was succesfully triggered by the incident.

Following the running of the playbook, it expected results were confirmed as follows; creation of a NSG security rule to block all inbound RDP connection to the Sales-VM. 

...closing the remote session of the attacker (Random-user) that connected with an untrusted IP. 


....sending an email notification to relevant members of the SecOps team for necessary actions.



## Conclusion

This project succesfully demonstrates the use of playbook in supporting incident response while managing security operations. It also provided an ample insights in designing the workflow of playbook in Azure Logic Apps for integration in Microsoft Sentinel. Overall, the project also elaborates the procedures to managing the life-cycle of real-world security incident from generation to closure.





## Past Project

* https://rhosinjay-cyb.github.io/Azure-Firewall/
* https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

