# Azure Virtual Machine Scale Sets (VMSS) Bootstrapping & Stress Testing Lab

![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Status](https://img.shields.io/badge/Deployment-Verified-success?style=for-the-badge)

A step-by-step implementation guide detailing the manual deployment, post-provisioning configuration using Custom Script Extensions, and workload stress testing across an Azure Virtual Machine Scale Set (`vmscaleset`) in the Azure Portal.

---

## 1. Lab Architecture & Provisioned Resources

All lab infrastructure was created inside resource group `lab-11042-2297488-0ae17ac7` in the `East US` region[cite: 8, 14]:


[Internet]
    │
    ▼
[Network Security Group: basicNsgvnet-nic01] ─── (Ports 22 & 80 Allowed)
    │
    ▼
[Virtual Network: vnet / Subnet: default]
    │
    ├──► [Storage Account: scalesetstoragedemo / Container: scalesetcontainer]
    │         └── commands.sh (NGINX Bootstrap Script)
    │
    ▼
[VMSS: vmscaleset (Standard_B2s, Ubuntu Linux)]
    ├── vmscaleset_0 (Port 80 Tested / Stress Tested)
    ├── vmscaleset_1 (Updated to Latest Model)
    └── vmscaleset_2 (Updated to Latest Model)
   Deployed Resource InventoryResource TypeResource NamePurposeConfiguration DetailVirtual machine scale set   vmscaleset   Scalable Workload FleetStandard_B2s (2 vCPUs), Ubuntu Linux, Uniform orchestration, 3 instances   Network security group   basicNsgvnet-nic01   Ingress Boundary ControlInbound Allow for SSH (22) and HTTP (80)   Storage account   scalesetstoragedemo   Script RepositoryStandard LRS, Blob container scalesetcontainer   Virtual network   vnet   Workload Network IsolationCIDR 10.0.0.0/16, Subnets default and snet-eastus-1   2. Step-by-Step Manual ImplementationStep 1: Configure Network Security Group RulesTo permit remote terminal administration and public web requests to the instances, inbound security rules were added to basicNsgvnet-nic01:   Navigated to Network security groups > basicNsgvnet-nic01 > Settings > Inbound security rules.   Added rule SSH: Port 22, Protocol TCP, Source Any, Destination Any, Priority 300, Action Allow.   Added rule port80: Port 80, Protocol Any, Source Any, Destination Any, Priority 310, Action Allow.   
Inbound security rules configured on basicNsgvnet-nic01.   Step 2: Prepare & Stage the Bootstrap ScriptAn automation shell script was prepared locally to update system repositories and install the NGINX web server non-interactively:
   
Preparing commands.sh in the code editor.   Navigated to the storage account scalesetstoragedemo.   Opened Data storage > Containers and navigated into scalesetcontainer.   Clicked Upload and uploaded commands.sh into the container.   
commands.sh hosted in container scalesetcontainer.   Step 3: Attach Custom Script Extension & Upgrade InstancesNavigated to Virtual machine scale sets > vmscaleset.   Under Settings > Extensions + applications, added the Custom Script Extension for Linux (Microsoft.Azure.Extensions.CustomScript) and linked it to commands.sh stored in scalesetcontainer.   Because the scale set upgrade policy is configured to Manual, selected all instances (vmscaleset_0, vmscaleset_1, vmscaleset_2) under Instances and clicked Upgrade to apply the extension model.   Confirmed all nodes transitioned to Latest model: Yes and Provisioning state: Succeeded.   
Instances upgraded and synchronized to the latest VMSS model.   Step 4: Validate Web Server DeploymentRetrieved the assigned public IP address (20.127.48.35) from the scale set network interface.   Entered http://20.127.48.35 into a browser to confirm that NGINX was installed and actively serving traffic.   
Verified active NGINX welcome page running on the VMSS instance.   Step 5: Execute CPU Stress Testing & Verify MetricsTo validate performance telemetry under load:Connected to instance vmscaleset_0 via SSH using the administrative user rootuser.   Updated local package indexes and installed the synthetic load generator stress:  

   
Installing stress package on vmscalese000000.   Executed the stress test generating 100 worker processes to consume CPU resources: 
   
Dispatching 100 CPU stress hogs.   Monitored Percentage CPU (Avg) on instance vmscaleset_0 in the Azure Portal, verifying sustained saturation past 85% to confirm Azure Monitor responsiveness.   
Azure Monitor CPU metric spike captured during the stress test.  
