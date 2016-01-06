---
layout: post
title: What are measures, dimensions and facts
tags: data warehouse
date:   2016-01-06 19:00:00
---

I did a bit of basic reading about measures, dimensions and facts. The most pithy explanation I have gathered is:-

- facts are tables, they record events, they are verb-like
- dimensions are tables, they record entities, they are noun-like
- dimension columns are columns in the fact table and are foreign keys into dimension tables
- measure columns are columns in the fact table and are things that matter to the business at hand
- dimensions columns are discrete
- measure columns are usually continuous

This build the foundation for data warehousing concepts. A number of variations on these concepts are also present but
this is the basic information necessary to navigate the world of data warehouses.
