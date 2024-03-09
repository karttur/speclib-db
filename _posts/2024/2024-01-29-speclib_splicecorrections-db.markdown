---
layout: post
title: Splicecorrection schema
categories: libspec-db-todo
excerpt: "Design and setup for schema model splicing, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-27 11:27'
modified: '2024-01-27 T18:17:25.000Z'
comments: true
share: true
---

## Introduction


This post contains the general design of the schema **splicecorrection** for the xspectre scan library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the scan schema

The purpose of the **splicecorrection** schema is ...



### splicecorrection



### Illustration of the splicecorrection schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-scan.png">
<img src="../../images/DBML_schema-scan.png"></a>
<figcaption>Scan DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for model pre-processing library'
}

Table modelsplicercorrection {
  splicecorrectionuuid UUID
  srccampaignuuid UUID
  dstcampaignuuid UUID
  srcsensoruuid UUID [pk]
  srcmuzzleuuid UUID [pk]
  dstsensoruuid UUID [pk]
  dstmuzzleuuid UUID [pk]
  dstmodeluuid UUID [pk]
  registerkey varchar(16) [pk]
  targetregistervalue real
  auxiliarykey
  auxiliaryvalue real
}

Table splicecorrection {
  campaignuuid UUID [pk]
  modeluuid UUID [pk]
  splicecorrectionuuid UUID [pk]
}
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the model per-processing schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "datatransform",
        "command": [
          "datatransformcode char(2)",
          "datatransform varchar(32)"
          "PRIMARY KEY (datatransformcode)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "datatransformcode",
        "command": [
          "datatransformcode char(2)",
          "datatransform varchar(32)"
          "PRIMARY KEY (datatransformcode)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "datatransform",
        "command": [
          "modeluuid UUID"
          "datatransformcode char(2)",
          "lambda real"
          "PRIMARY KEY (modeluuid)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "scattercorrection",
        "command": [
          "modeluuid UUID"
          "scattercorrection varchar(32)",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "meancentring",
        "command": [
          "modeluuid UUID"
          "meancentring varchar(32)",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelpreprocess",
        "table": "meancentring",
        "command": [
          "modeluuid UUID"
          "meancentring varchar(32)",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    },



  ]
}
```

### Prepartion codes

TO BE ADDED
