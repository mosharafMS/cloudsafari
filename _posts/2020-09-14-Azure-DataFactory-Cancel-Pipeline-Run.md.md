---
layout: post  
title:  "Cancel Data Factory Pipeline Run "  
tags:  [ Azure, Data Factory, Pipeline, Run, Cancel ]  
featured_image_thumbnail: assets/images/posts/2020/ADF-thumbnail.jpg 

date: 2020-09-14
category: Data-Engineering

---

In some cases you want to end the [Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/) (ADF) pipeline execution based on a logic in the pipeline itself. For example, when there's no record coming from one of the inputs datasets then you need to fail quickly to either reduce cost or to avoid any logical errors. 

The challenge is there's no activity in ADF that cancels execution. In this case, the [web activity](https://docs.microsoft.com/en-us/azure/data-factory/control-flow-web-activity) comes handy 

Let's take a quick example, in the picture below, a simple logic pipeline that sets a variable to true and then based on the value of the variable, it has *If Condition* activity to cancel the pipeline execution when it's true (of course it's always true in this example, but you get the point)
![simple logic pipeline](/assets/images/posts/2020/adf-logic.png)

Inside the true branch of the *If Condition* add *Web* activity (under general) 
![Web Activity](/assets/images/posts/2020/adf-web-activity.png)

### Now what are we trying to do?
Since there's no activity then we need to call the ADF REST API which is part of the Azure Resource Manager (https://management.azure.com) .
The API we are interested in is the [Cancel Pipeline Run API](https://docs.microsoft.com/en-us/rest/api/datafactory/pipelineruns/cancel) 

The API format 
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories/{factoryName}/pipelineruns/{runId}/cancel?api-version=2018-06-01
Luckily the {factoryName} and the {runId} can be obtained using ADF functions during runtime (with a catch) using `pipeline().DataFactory` and `pipeline().RunId` respectively 

**What's the catch?** 
The `RunId` doesn't work when execute the pipeline under debug mode. You need to trigger the pipeline to get the correct `RunId`
That makes the REST call like this 

    @concat('https://management.azure.com/subscriptions/***subscriptionID***/resourceGroups/***resource group name***/providers/Microsoft.DataFactory/factories/',pipeline().DataFactory,'/pipelineruns/',pipeline().RunId,'/cancel?api-version=2018-06-01')

**How about authentication?**
Correct, ARM REST API calling can be daunting because of the oauth authentication workflows. Fortunately the *Web* activity supports [*Managed (Service) Identity*](https://docs.microsoft.com/en-us/azure/data-factory/data-factory-service-identity) . In nutshell, every Data Factory instance has a service identity created in Azure AD to be used by this instance. 

The settings page should look like this 

![Web Activity Settings](/assets/images/posts/2020/ADF-web-activity-settings.png)

Note that the `resource` is the resource that you want to connect to which is the Azure Resource Manager (ARM) API ==> https://management.azure.com 

If you run the pipeline this way, probably you will get access denied error because this managed identity doesn't have permissions to stop the pipeline run for this data factory. To grant this permission, assign the managed identity the Data Factory contributor Role (least privileged) or Contributor role (more privileged) 

> Note:
> > The Managed Identity name is the same name as the data factory
> > There's optional parameter to this API to cancel recursively so if the pipeline calls another pipeline, the called pipeline will be cancelled as well 


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDc2NjgwODksMTg4NDMxMDgzLC0xOD
QwMDQ4NDc2LDcxMzMyNDkxOCwxOTY3NTg2OTU5LDkwNjYyNDE2
OV19
-->