---
layout: post
title: Schema modelvalidations
categories: libspec-db-todo
excerpt: "Design and setup for schema model validation, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-02-08 11:27'
modified: '2024-02-08 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

The **modelvalidations** schema saves the results of deployed Machne Learning (ML) models. The actual model settings and results are saved as postgreSQL json objects.

evaluation and definition of Machine Learning (ML) models using spectra and spectra derivates as covariates and aiming to predict any physical, chemical and biological property of the substance (target feature) under stury. The **modelsetups** schema itself contains tables for:
- metadata defining a model, and
- process steps to include for setting up model development, testing and validation.

Tables defining the process parameters and algorithms to apply for e.g. covariate correction and selection, parameter settings for model testing and validation, plotting and hyperparameter tuning are defined in separate schemas (see below). The **modelsetups** schema, however, contains the boolean (yes/no) arguments determining whether or not to include a particular process or algorithm in the process chain.

This post contains the general design of the schema **modelsetups** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the model validation schema

The purpose of the **modelvalidations**

### Illustration of the modelvalidations schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-modelvalidations.png">
<img src="../../images/DBML_schema-modelvalidations.png"></a>
<figcaption>Modelvalidations DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for model validation library'
}

Table modelvalidation {
  modeluuid UUID [pk]
  featureimportance boolean
  modelvalidation boolean
}

Table featureimportance {
    modeluuid UUID [pk]
    reportmaxfeatures smallint
    permutationrepeats smallint
    coefficientimportance boolean
    permutationImportance boolean
}

Table modelvalidation {
    modeluuid UUID [pk]
    traintest boolean
    kfold boolean
}

Table traintest {
    modeluuid UUID [pk]
    testfraction real
}

Table kfold {
    modeluuid UUID [pk]
    nfolds smallint
}

Table modelplot {
    modeluuid UUID [pk]
    plotcode char(1)
}


```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the scan schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "subsample",
        "command": [
          "sampleuuid UUID",
          "subsample char(2)",
          "scandatetime timestamp",
          "PRIMARY KEY (sampleuuid,subsample)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "spectramode",
        "command": [
          "mode varchar(2)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "scanspectra",
        "command": [
          "sampleuuid UUID",
          "subsample char(2)",
          "scanuuid UUID",
          "spectromuzzleuuid UUID",
          "prepcode char(2) [NOT NULL]",
          "mode varchar(16) [NOT NULL]",
          "nafreq real",
          "negfreq real",
          "extfreq real",
          "PRIMARY KEY (sampleuuid, subsample, prepcode, mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "reflectance",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "transmissivity",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "fluoresence",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "raman",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "scanprobe",
        "command": [
          "sampleuuid UUID",
          "subsample varchar(8)",
          "probeuuid UUID",
          "prepcode char(2)",
          "mode varchar(16)",
          "scanuuid UUID",
          "PRIMARY KEY (sampleuuid, subsample, prepcode, mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "proberecord",
        "command": [
          "scanuuid UUID",
          "registerkey varchar(16)",
          "registervaluemean real",
          "registervaluestd real",
          "PRIMARY KEY (scanuuid, registerkey)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "spectramode",
        "command": [
          "mode varchar(2)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (mode)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
