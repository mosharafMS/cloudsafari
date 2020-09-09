---
layout: post  
title:  "Cancel Data Factory Pipeline Run "  
tags:  [ Azure, Data Factory, Pipeline, Run, Cancel ]  
featured_image_thumbnail: assets/images/posts/2020/ADF-thumbnail.jpg 

date: 2020-09-14
category: Data-Engineering

---

In some cases you want to end the [Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/) (ADF) pipeline execution based on a logic in the pipeline itself. For example, when there's no record coming from one of the inputs datasets then you need to fail quickly to either reduce cost or to avoid any logical errors. 

The challenge is there's no activity in ADF that cance

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjYzNzcwOTE3LDkwNjYyNDE2OV19
-->