---
layout: post
title: Schema hyperparametertuning
categories: libspec-db
excerpt: "Design and setup for schema scan, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-02-06 11:27'
modified: '2024-02-06 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

In Machine Learning (ML) modeling, a hyper-parameter controls the learning process and is not itself trained. Some hyper-parameters does not affect the training as such, only the speed or efficiency of the learning. Among the latter are for instance the number of kernels that are employed when running a regressor. If you apply hyper-parameter tuning when formulating a ML model, the search space for the parameter tuning will be saved in the schema **hyperparametertuning**.

This post contains the general design of the schema **hyperparametertuning** for the xspectre modeling library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the hyperparametertuning schema

The purpose of the **hyperparametertuning** schema is to store successful settings for hyper-parameter tunings. The schema contains three tables:

- hyperparametertuning,
- randomtuning, and
- exhaustivetuning.

### Illustration of the hyperparametertuning schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-hyperparametertuning.png">
<img src="../../images/DBML_schema-hyperparametertuning.png"></a>
<figcaption>HyperparametertuningDBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for hyperparamtuning library'
}

Table modelsetups.spectralmodel {
  modeluuid UUID
  morecolumns char(1)
}

Table hyperparametertuning {
  modeluuid UUID [pk]
  fraction real
  n_itersearch smallint [default:6]
  n_bestreport smallint [default:3]
  randomtuning boolean
  exhaustivetuning boolean
}

Table randomtuning {
    modeluuid UUID [pk]
    tuningparameter varchar(32) [pk]
    minumum real [default:0]
    maximum real [default:0]
    choicelist TEXT[]
    numberlist real[]
    nestednumberlist real[][]
    trueorfalse boolean [default:false]
}

Table exhaustivetuning {
  modeluuid UUID [pk]
  tuningparameter varchar(32) [pk]
  minumum real [default:9]
  maximum real [default:0]
  choicelist TEXT[]
  numberlist real[]
  nestednumberlist real[][]
  trueorfalse boolean [default:false]
}
Ref: modelsetups.spectralmodel.modeluuid - hyperparametertuning.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - randomtuning.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - exhaustivetuning.modeluuid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the hyperparametertunings schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "hyperparametertunings",
        "table": "hyperparametertuning",
        "command": [
          "modeluuid UUID",
          "fraction real",
          "n_itersearch smallint DEFAULT 6",
          "n_bestreport smallint DEFAULT 3",
          "randomtuning boolean",
          "exhaustivetuning boolean",
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
        "schema": "hyperparametertunings",
        "table": "randomtuning",
        "command": [
          "modeluuid UUID",
          "tuningparameter varchar(32)",
          "minumum real DEFAULT 0",
          "maximum real DEFAULT 0",
          "choicelist TEXT[]",
          "numberlist real[]",
          "nestednumberlist real[][]",
          "trueorfalse boolean DEFAULT false",
          "PRIMARY KEY (modeluuid, tuningparameter)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "hyperparametertunings",
        "table": "exhaustivetuning",
        "command": [
          "modeluuid UUID",
          "tuningparameter varchar(32)",
          "minumum real DEFAULT 0",
          "maximum real DEFAULT 0",
          "choicelist TEXT[]",
          "numberlist real[]",
          "nestednumberlist real[][]",
          "trueorfalse boolean DEFAULT false",
          "PRIMARY KEY (modeluuid, tuningparameter)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
