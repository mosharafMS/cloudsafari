---
layout: post  
title:  "Call Databricks API from Logic Apps"  
tags:  [ Databricks ]  
featured_image_thumbnail: /assets/images/posts/2020/databricks-api-thumbnail.png
date: 2020-10-26
category: Data-Platform
---

# Call Databricks API from Logic Apps

Recently I needed to help a customer to call Databricks API and since there are many ways to do this I must start by scoping the scenario 

- This is Azure Databricks not Databricks on another cloud provider. 
- Authentication can be done by 3 ways 
  - Azure Databricks [Personal Access Token](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/api/latest/authentication)
  - Using Azure AD access token [for a user](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/api/latest/aad/app-aad-token#use-token) so we need to impersonate a user access to access Databricks
  - Using Azure AD access token [for service principal](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/api/latest/aad/service-prin-aad-token#use-token)
- In this scenario we chose using service principal because it will be used by a service **because** I'd like to keep all the identities centralized in Azure AD for easy management. 
- The APIs that we needed are to list running clusters and terminate them. auto-terminate wasn't an option because of some restrictions related to this implementation. 



### The chosen service to run the automation

We chose [Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview) for simplicity however all what we are doing is calling REST APIs so whether it's logic apps, Function app, automation runbook or any other service hosted inside a VM it's the same concept 



### The workflow

The workflow I'm using as illustrated by the diagram below 

![Calling Databricks API workflow](/assets/images/posts/2020/databricks-api-calls.png)

- [ ] Get Access token for the Databricks login app
- [ ] Get Access token for the Azure management endpoint 
- [ ] Use the two tokens when calling any Databricks API

**But why two access tokens?**

​          Because Databricks is very well integrated into Azure using the Databricks resource provider, some APIs requires Azure management (think of anything you can change from the Azure portal) and some require login to the Databricks workspace (i.e listing and updating clusters) however the APIs designed in a way to require both tokens for all of them (or at least up to my knowledge). For that we have to do two API calls to the Azure AD login endpoint



**What's he difference between the two API calls?**

Both are identical except for the resource to get the access token to. In Azure AD, you must specify why do you need access token. Meaning what resource you want to access by this token so Azure will get you a token <u>*only*</u> for this service. For the Databricks login app, you must use the guid "**2ff814a6-3304-4ab8-85cb-cd0e6f879c1d**" which if you navigate to Azure portal and searched for this id, you will find it associated with enterprise app named AzureDatabricks. This is the app representing Databricks to facilitate login to the workspace



###  Collecting the requirements

**App info**

To start getting the access tokens, we need the service principal info. Provided that you have app registration already created in Azure AD. The process to provision he service principal is [documented well in the docs](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/api/latest/aad/service-prin-aad-token#--provision-a-service-principal-in-azure-portal) so no need to repeat it 

> In this context we can use Azure AD app & service principal interchangeably. However they are not both the same. App is one instance that can be shared across multiple directories (Databricks login app is example ) and the service principal is the representation of this app inside the directory. When we authorize, we authorize the service principal not the app. 

![App Info page in Azure AD](/assets/images/posts/2020/AAD-App-info.png)

From the App info page, collect 

- Client ID
- Tenant ID
- Then navigate to the *Certificates & Secrets* page from the left navigation bar and generate a secret. 

Consider all these information as secrets and you should keep them safely in a keyvault or a similar secret management solution. The logic app I'm including with this article expect all these as input so it doesn't save or retrieve secrets. I made it this way to be re-usable. More to this later 

**Azure Resource info**

Databricks workspace is an Azure resource, you need to collect 

- subscription id
- resource group name
- workspace name 

We will use them later inside the log app to generate the resource id

**Databricks instance**

All the Databricks URLs are using the instance name which is what comes before *azuredatabricks.net* in the URL when you login to the Databricks UI. It's auto generated and usually starts with *adb-* then numbers



### The logic app

The complete code of the app at the end of this article. I'll go through the main steps with some description 

![logic app trigger](C:\Users\mosharaf\source\repos\cloudsafari.ca\assets\images\posts\2020\logic-app-trigger.png)

The logic app is triggered by an http trigger. This way I can call it from another logic app that fetch the secrets from key vault. So I can reuse and share this one without worrying about secret management



![Access token for databricks login app](C:\Users\mosharaf\source\repos\cloudsafari.ca\assets\images\posts\2020\logic-app-access-token-dbricks.png)

This is the first access token. we get Azure AD access token for **the Databricks login app** that will be used to access the Databricks instance

This step is followed by a step to parse the return json to get the access token out of it. 

Next is to issue almost identical REST API call to authenticate with only one difference is the resource=**https://management.core.windows.net/**



![list clusters API](C:\Users\mosharaf\source\repos\cloudsafari.ca\assets\images\posts\2020\logic-app-list-clusters.png)



This is the first API call to Databricks. There's another one later in the app but the principal is the same so I'll explain here this one only. 

**URL:** The URL is on the format **https://<Databricks Instance Name>..azuredatabricks.net/api/2.0/clusters/list** so I concatenated the input parameter into the URL

**Headers:**

​		**Authorization:** the concatenation of the keyword **Bearer** and the access token we got for the <u>Databricks login app (where the resource is the app id)</u> 

​		**X-Databricks-Azure-SP-Management-Token:** The access token (without Bearer keyword) of the Azure management endpoint

​		**X-Databricks-Azure-Workspace-Resource-Id:** The resource Id of the workspace, I used the input parameters of the workspace name, resource group name and subscription id to create it. 





And here's the complete code of the logic app



{% gist cbff418e7158a499d5fd6b4e389063ab %}