
# Azure Databricks Networking Deep dive

This is an updated version of my article at [Medium.com](https://medium.com/cloudsafari-ca/azure-databricks-deployments-issues-3195ea8c7f56) originally written on December 2019 as some changes happened since then 

Azure Databricks  [workspace](https://docs.microsoft.com/en-us/azure/databricks/workspace/) is a code authoring and collaboration workspace that can have one or more Apache Spark clusters. So as a prerequisites to create the cluster, there has to be a  [Virtual Network (VNET)](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)  to have the machines attached to itd. That’s why in the managed resource group, if you don’t choose to use your own VNET, there will be a VNET created for you.

Many organizations have restrictions about the VNET creation and prefer to integrate the Databricks clusters into their network infrastructure, that’s why Azure Databricks now supports VNET injection. VNET injection enable you to use an existing VNET for hosting the Databricks clusters.

The  [docs](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject)  are listing the benefits & the requirements of these VNETs so I won’t list all of them here however the most important ones

-   Have to be in the same subscription
-   **Address space:**  A CIDR block between /16 — /24 for the virtual network and a CIDR block up to /26 for the private and public subnets
-   Two subnet per workspace
-   Can’t use the same subnet by two workspaces --> That means every time there's a new workspace, there's quite a checklist to go through which makes the idea of hosting multiple projects/departments on the same workspace a favorable idea but that's the topic of another article. 
- ----------------------------------------------
**But first, what’s deployed inside the customer’s VNET? is Azure Databricks entirely deployed there?**
No, the control plane and web UI are always deployed in a Microsoft-managed subscription. That’s why the Azure Databrisk UI is always https://< Azure Region >.azuredatabricks.net

![Azure Databricks Network Architecture](/assets/images/posts/2020/dbricks-vnet-architecture.png)

During workspace deployment, there’s no clusters created yet. So during deployment, Databricks would insure that the minimum requirements for the clusters to be successfully deployed are met.

**What happens during deployment time?**

![Azure Databricks Creation Flowchart](/assets/images/posts/2020/dbricks-creation-flowchart-network.png)

-   Delegation to Microsoft.Databricks/workspaces is made on each subnet. This delegation will be used by the service to create a Network Intent Policy. Azure Network Intent Policy is a feature used by some Microsoft first party providers like SQL Managed Instance. It’s not available for customers’ use as there’s no public docs for it. In the case of Databricks, it’s used to maintain the NSG rules added by the workspace creation. If you try to delete one of the rules, you will get an error message because of the Intent Policy.
-   Check if there’s already an NSG attached to the subnet. If not then the creation of the workspace will fail. In the portal experience, the ARM template generated by the portal is avoiding the failure by creating an NSG and attach it. It will create only one NSG for both private & public subnets. The rules are identical anyways even if you provided two NSGs, you will find the same set of rules.
-   If there’s NSG attached, new rules will be added and protected by the Intent Policy so it can’t be changed or deleted.

I’ve created  [a sample ARM template on github](https://github.com/mosharafMS/ARM-Templates/tree/master/Databricks/vnetInjection)  that deploys Databricks workspace with VNET integration but you have to setup the delegation and attache NSGs to the subnet before you deploy the template

-----------------------------------------

**Why having two subnets?**
Each machine in the Databricks cluster has two virtual network cards (NIC), one with private IP only attached to the private subnet and one with both private & public IPs attached to public subnets. The public IPs used to communicate between your cluster and the control plane of Azure Databricks plus any other data sources that might reside outside your Vnet. All the inter-cluster communication happens on the
 **Are the subnets exclusively used by the workspace clusters only?**
No, you can use subnets that has NICs already attached to them. But you can't have two different workspaces on the same subnet

## Integrating Azure Databricks workspaces with firewalls
On of the main reasons to have Azure Databricks workspaces integrated with VNET is to utilize your existing firewalls. The workflow to do that is

 1. Create and assign Route Tables that  will  force the traffic from
    Azure Databricks to be filtered by the firewall first.
 2. In the firewall you add rules to allow the traffic needed for your workspace

*The traffic coming to the public IPs of the workspace clusters, doesn’t pass on the firwall and if your routing route all 0.0.0.0/0 to the firewall that means the firewall will only see the return of the traffic and for any stateful firewall like Azure Firewall, it will drop the traffic. You have to add exception of the routing for the control plane. The screenshot below shows this problem. For a complete list of the IPs per region, refer to the* [docs](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/udr)

![Asymmetric traffic](/assets/images/posts/2020/assymetric-network.png)

Because of how stateful firewalls work , the routing table should avoid routing *ALL* the traffic but to exclude the traffic going to the control plane and the web app of Databricks
 
![Sample Routing Table](/assets/images/posts/2020/routing-table.png)

In my testing, I used [Azure Firewall](https://docs.microsoft.com/en-us/azure/firewall/overview) and I'll list all the Network Rules and Application Rules I added to have a successful cluster creation and running sample notebook. 

## Documented Rules
### Control plane NAT and Webapp IP addresses
Per each Azure region that has Databricks enabled in, there are two IP ranges, one for the control plane and one for the webapp 

## Special un-documented domains
During my testing with Azure Firewall & Databricks, I found that the docs didn’t cover all the FQDNs that are requested by my cluster. From my testing I found out these extra ones
-   Ubuntu updates → *.ubuntu.com
-   snap packages → *.snapcraft.io
-   terracotta → *.terracotta.org
-   cloudflare → *.cloudflare.com

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTYyMjE0NTE0LC0xNjk3NTcwODc0XX0=
-->