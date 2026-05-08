## Project Title

Automating incident response with playbooks

## Objective

The aim of this project is to design an Azure Logic Apps playbook and integrate it with Microsoft Sentinel via automation rule to facilitate automated response to security incidents in Microsoft Sentinel, thereby supporting the effective management of security operations (SecOps).

## Tools Used

Microsoft Sentinel, Microsoft Defender Portal, Azure Logic Apps


## Lab Setup
* Creation of analytics rule
* Designing the workflow of the playbook with Azure Logic Apps
* Creation of automation rule (Integration of playbook with incident)
* Testing of the playbook



## Background Information
This project marks the continuation of my project on managing security operation with tools from the Microsoft security stack. Prior to the commencement of this project, the following resources and services (Azure Bastion, Project-BST; Virtual Machine , Sales-VM; Virtual Network, Project-vnet; Subnet, Sales-vnet & AzureBastionSubnet; Microsoft Sentinel integrated-Log Analytics Workspace, Project-workspace; all in the same resource group, Project-RG) have already been deployed and will be used during the testing phase of this project. Additionally, the diagnostic settings of Azure Bastion (Project-BST) has also already been enabled and its events logs are being sent to the Microsoft Sentinel integrated-Log Analytics Workspace (Project-workspace) for analysis.

To put this project into context, the sales team of a company has a virtual machine (Sales-VM) which host critical applications and data, and would want to enhance its security. The team has approved Azure Bastion and one particular IP address for remote connection to the VM. The team wants every other IPs to be deemed untrusted and such connections should be blocked. Hence, the security solution for the sales team is the core of this project. 

Initially, each of the IPs that are used for the remote connection could be monitored from the event logs stored in MicrosoftAzureBastionAuditLogs table in the Project-workspace.
To provide a solution to the sales team security challenge, an analytics rule will be created to detect untrusted IPs, the playbook workflow will be designed, and the automation rule attached to the analytics rule will be utilized to trigger the playbook. The main actions of the playbook is to block the inbound connection to the VM, particularly RDP traffic, once the attack is detected. However, the blocked traffic will only prevent future remote connection to the VM while the attacker's remote session remains active. The next action is to identify the attacker's remote session among all active remote sessions and shut it off. The last action of the playbook is to send an email notification about the result of the playbook to relevant members of the security operations team for neccessary actions.

The creation of the analytics rule and the design of the workflow in the playbook will be discussed in the next section, while the testing of the playbook and conclusion follows respectively.

## Steps Taken

The project was accomplished in the following order

### Creation of Analytics Rule

The project commenced with the creation of an analytics rule (scheduled) to detect remote connections to the Sales-VM via Azure Bastion with an untrusted IP.

![image](Images/A.Rule.png)

### Creation of the Azure Logic App playbook

The playbook is created with Microsoft Sentinel incident as the trigger. While creating the playbook the subscription and resource group is specified alongside the tier of the Azure Logic App.

![image](Images/CR.Playbook.png)

### Designing the workflow of the playbook

Following the creation of the playbook, the the workflow of the playbook from trigger to the last action was designed with the logic app designer blade.

![image](Images/Workflow.png)

The trigger for the workflow is Microsoft Sentinel Incident because the playbook is designed to run when an incident is created in Microsoft Sentinel.

To collect the entities in the incident, these variables of array data type are initialized to collect and store the entities. 

Note: Entity mapping is the responsibility of the security team while creating the analytics rule, mapping the right entity to the analytics rule assists in enriching incident investigation.

![image](Images/INI.Variables.png)

The for each loop has two seperate conditions which were used to collect the entities. The first conditons appends IPs to the array IPList if the entity kind equals Ip while the under condition appends Username to the array UserList if the entity kind equals Account.

![image](Images/For.Each.png)

The above actions ONLY implements the collection of entities from the security incident.

The next action was used to create a network securtiy group (NSG) security rule to block all inbound connection to the Sales-VM including incoming RDP traffic. The Azure Resource Manager connector is used for this action and 'Create or update resource' was choosen among the available options. The parameters were filled accordingly, resource explorer would be quite helpful in getting the short resource id. Location and properties were selected among the advanced parameters and equally specified as well. The values in the properties field will now be used to create a NSG security rule by updating the Sales-VM-nsg which will block inbound connections to the Sales-VM. Blocking the inbound traffic will prevent the attacker from reconnecting.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a network contributor role at the NSG of the virtual machine (Sales-VM-nsg) enforcing the principle of least privilege.

![image](Images/Blk.NSG.png)

The next action in the workflow is the closing of the attacker's remote. This action also ensured that only the malicious session was closed while other sessions were skipped preventing the disruption of production operation. This action was implemented with the HTTP connector, which sent a web request to the API of Azure Resource Manager (ARM), the ARM validated the request and allowed the runCommand service to utilize the Azure VM agent to exexute the powershell script specified in the body of the HTTP request, basically to close the attacker's remote session.

Note: For the Playbook to complete this action the managed identity of the playbook needs to be assigned a Virtual Machine contributor role at the scope of the resource, Sales-VM.

![image](Images/Close.MRDP.png)

The last of the actions on the workflow was to send an email notification to relevant members of the security operations team for incident review and other neccessary actions. Aside alerting the team, the email also provide a summary of the attack at a glance with details that includes the entities extracted from the security incident.

![image](Images/Send.Email.png)

### Integration of the playbook with Microsoft Sentinel
To integrate the newly created playbook with Microsoft Sentinel, an automation rule was created. There were two actions in the automation rule, the first one was to assign the incident to a member of the SecOps team while the other action was to run the playbook.

![image](Images/Aut.Rule1.png)
![image](Images/Aut.Rule2.png)

A major prequisite that was implemented for seamless interaction between Microsoft Sentinel and the playbook was to configure the playbook permission of Microsoft Sentinel at resource group level. This configuration automatically assigns the Microsoft Sentinel Automation Contributor role to Azure Security Insights (service principal) which gives Microsoft Sentinel the permission to trigger any playbook in that resource group.

![image](Images/Playbk.perm.png)

## Testing of the Playbook

The Sales-user account was used to logon to the sales-VM via Azure Bastion with the trusted IP. Then a new user (Random-user) was created  on the Sales-VM and it was added to the remote desktop users group to allow the Random-user to able to logon remotely. 

![image](Images/CR.User.png)

Note: The new user (Random-user) was only created to effectively demonstrate the testing of the playbook.

![image](Images/Random_user.png)

To simulate an attack scenario, the Random-user account was used to logon to the Sales-VM remotely via Azure Bastion but with an untrusted IP address. 

![image](Images/LogontoVM.png)

![image](Images/ConnecttoVM.png)

The detection of the usage of an untrusted IP for connection to the Sales-VM led to the firing of a Microsoft Sentinel alert and an incident thereafter. 

![image](Images/INCIDENT2.png)

While reviewing the incident page, it was observed that the automation rule had assigned the incident to a member in the SecOps team.

Also, the review of the incident page showed that the playbook was succesfully triggered by the incident.

![image](Images/Ran_success.png)

Following the running of the playbook, it expected results were confirmed as follows; creation of a NSG security rule to block all inbound RDP connection to the Sales-VM. 

![image](Images/NSG_success.png)

...closing the remote session of the attacker (Random-user) that connected with an untrusted IP. 

![image](Images/Closed_success.png)

....sending an email notification to relevant members of the SecOps team for necessary actions.

![image](Images/email.png)

The post-review of the security incident, includes the following:
* deleting the NSG rule created by the playbook action to allow remote connection to the VM

![image](Images/NSG_Delete.png)
  
* classifying the security incident

![image](Images/manage_incident.png)
  
*  and closing of the incident

![image](Images/resolved.png)

![image](Images/evidence_resolved.png)

## Sensitivity Analysis

Further tests were implemented to test the performance of the playbook. In this case, another user account (SecondRandom-user) was created on the Sales-VM. Afterwards, both user accounts (Random-user & SecondRandom-user) were used to logon to sales-VM via Azure Bastion with both users using different untrusted IPs. 

![image](Images/extra3.png)

![image](Images/IPs.png)

![image](Images/Users.png)

Similarly, the attack was detected, and it led to a security incident. The incident page dispalys the Accounts and IPs involved in the attack. Each of the sessions were also closed, and all the entities related to the attack were extracted and sent with the email notification. In cases where the attack is multiple and simultaneous with different IPs, every entity will be extracted by the playbook just as seen in the incident page.

![image](Images/email2.png)

## Conclusion

This project succesfully demonstrates the use of playbook in supporting incident response while managing security operations. It also provided an ample insights in designing the workflow of playbook in Azure Logic Apps for integration in Microsoft Sentinel. Overall, the project also elaborates the procedures to managing the life-cycle of real-world security incident from generation to closure.

## Past Project

* https://rhosinjay-cyb.github.io/Azure-Firewall/
* https://rhosinjay-cyb.github.io/Microsoft-Sentinel/

