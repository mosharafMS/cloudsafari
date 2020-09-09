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

**Now what are we trying to do?** 
Since there's no activity then we need to call the ADF REST API which is part of the Azure Resource Manager (https://management.azure.com) .
The API we are interested in is the [Cancel Pipeline Run API](https://docs.microsoft.com/en-us/rest/api/datafactory/pipelineruns/cancel) 



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NDAwNDg0NzYsNzEzMzI0OTE4LDE5Nj
c1ODY5NTksOTA2NjI0MTY5XX0=
-->