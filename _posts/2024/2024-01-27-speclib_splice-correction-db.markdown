---
layout: post
title: Model preprocessing schema
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

In the xSpectre database the spectral data pre-processing steps to apply for any particular analysis or modelling are stored in the table **modelpreprocessing**.

This post contains the general design of the schema **modelpreprocessing** for the xspectre scan library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the scan schema

The purpose of the **splicecorrection** schema is to store the combination of pre-processing steps defined for all data analysis and modeling. The principal pre-processing methods include:



### splicecorrection

Data transformations implemented in the xspectre modeling environment include:

- logarithmic transformation
- square root transformation
- inversion
- yeo-johnson transformation

Of the above transfroamtions, only the yeo-johnson requires parameterisation; this parameter is saved in the datatarform table for models applying the yeo-johnson transformation.

### scattercorrection

TO BE ADDED

### meancentring

TO BE ADDED

### Splicecorrection




### Illustration of the model pre-processing schema

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
  splicecorrectionuuid char(36)
  srccampaignuuid char(36)
  dstcampaignuuid char(36)
  srcsensoruuid char(36) [pk]
  srcmuzzleuuid char(36) [pk]
  dstsensoruuid char(36) [pk]
  dstmuzzleuuid char(36) [pk]
  dstmodeluuid char(36) [pk]
  registerkey varchar(16) [pk]
  targetregistervalue real
  auxiliarykey
  auxiliaryvalue real
}

Table splicecorrection {
  campaignuuid char(36) [pk]
  modeluuid char(36) [pk]
  splicecorrectionuuid char(36) [pk]
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
          "modeluuid char(36)"
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
          "modeluuid char(36)"
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
          "modeluuid char(36)"
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
          "modeluuid char(36)"
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
