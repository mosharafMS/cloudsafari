---
layout: post  
title:  "Data Analytics Basics - Critical Review"  
tags:  [ Data-Lake, Data-Warehouse, Data-Hub ]  
featured_image_thumbnail: /assets/images/posts/2021/analytics-thumbnail.jpg
featured: true 
date: 2021-02-13
category: Data-platform
---

I had the pleasure recently to deliver a session at the [Worldwide Software Architecture Summit](https://architecture.geekle.us/). I see a rise of more virtual-only events phenomenon even beyond Covid as many now realized that the value/cost balance for these events is very positive. 

 In this post, I will try to summarize the talk in few paragraphs. 

The talk -as implied by the title- is trying to discuss the basics but away from the restraints of what's hot these days. I also wanted to be vendor-neutral as the conference track's objective to discuss the architecture away from the vendor specifics. These are the main points 
<!--more-->
- When a customer lists the data sources that they are interested in including in their data warehouse, they typically ignore two types of data 
  - Dark Data: That's the data collected from websites logs, telemetry and not analyzed for long term
  - External (contextual) Data: That's the data that might be collected *about* the business not by the business itself. Examples: social media mentions to the company or the products, analysts/news agencies publishing about the company or the products 
- Data warehouse *is a logical architecture* not a <u>product</u>. Many products can act as a data warehouse. I gave an example of a SQL Database used as a data warehouse when MPP architecture is not needed 
- Data Lake is a *concept* that -to be realized- requires many products from the same or different vendors to be used together. Not one product currently in the market that I'm aware of can create a complete data warehouse. 
- Data Lake storage is just storage. You can think of it as your laptop drive. you save files on it. These files can be of a special type that represent data (csv, parquet, avro...) or just files like pictures. The repeated marketing messages of storing structured, un-structured and semi-structured data gave many people the impression that there is some magic there
  - Before you passionately disagree, yes there are great technologies packed into the storage to make it accessible to multiple nodes/machines with multiple threads accessing the same file. That and other differences related to availability & disaster recovery are masked by the cloud providers and since my scope is cloud so I wont go into the details of making a hard drive => data lake :)
- Data Lake storage doesn't have indexes or statistics by itself. You need to have a big data cluster like spark to load the files into data structured that are indexed. New technologies like [Delta Lakes](https://delta.io/) give extra layer of transactional consistency to the parquet files but still the performance of the relational databases is superior to the lakes
- Data Hub is a *logical architecture* which enables data sharing. I see this ignored/avoided at many customers discussions mainly because the data team is mainly consists of data engineers & data scientists and neither of them -typically- have experience in Enterprise integration APIs/service bus. In my opinion that shouldn't hinder the architect from considering this as a logical component that can be physically satisfied by many implementations. A csv file on a blob/file container can be your data hub. I see SFTP in many cases upgraded to be a data hub. All these are accepted to some extend as long as in the original blueprint data hub was a basic component and didn't just put there without holistic understanding of its function. 
- Relational Data Warehouses are not dead and according to Gartner many organizations are planning to use it. I see dependency on **only data lake** is un-natural and will cause the failure of the data analytics project when trying to force it on the data team. 

These are the main messages and I hope I could convey them to you to the best of my knowledge. [The slide deck are here](/assets/docs/Data Analytics Architecture.pdf) 

I'm also adding the recorded session (the recording was not live) here

![]({https://www.youtube.com/watch?v=XfTJwdPGjko})



