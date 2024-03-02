---
layout: post
title: Data Pipeline with Google Cloud Platform
subtitle: Cloud Composer pipeline with Delta Lake architecture on Big Query
tags: [gcp, big-query, python, composer, sql]
author: Sarah Silva
---

The main object of this project was to collect data from an API, store it on a database so that we could perfom a couple of queries on them.
To solve this issue, the following architecture was implemented: 

![Architecture](../img/arq.png)