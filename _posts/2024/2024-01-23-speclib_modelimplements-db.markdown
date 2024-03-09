---
layout: post
title: Schema modelimplements
categories: libspec-db-skip
excerpt: "Design and setup for schema modelimplements, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-21 11:27'
modified: '2024-01-21 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

The **modelimplements** schema stores the settings and performances of models that are accepted as deployable. This post contains the general design of the schema **modelimplements** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the modelimplements schema

The purpose of the **modelimplements** schema is to record:
 1. the covariate and parameter settings of models accepted from the formulation stage,
 2. model statistical performance results,
 3. the model itself.

The tables of the schema include:

- modelparameters (hyperparameter settings for recorded models, reporting of feature importance and model validation),
- parametertypecode (support list with codes of hyperparameter types),
- modelcovariates (final model selected covariates),
- modelkfoldresult (model statistical performance and plots for best fitted models),
- modeltraintestresult (model statistical performance and plots for best fitted models), and
- pubmodel (models published for independent prediction).

Note that many other parameter settings are saved for the same model (modeluuid) in the [**modelsetups**](../speclib_modelsetups-db/) schema, including all the spectra pre-processing steps.

### Illustration of the modelimplements schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema_modelimplements.png">
<img src="../../images/DBML_schema_modelimplements.png"></a>
<figcaption>Modelimplements DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra model library'
}

Table modelsetups.spectralmodel {
  modeluuid UUID
  morecolumns char(1)
}

Table modelparameters {
  modeluuid UUID [pk]
  hyperparameter varchar(32) [pk]
  parametertypecode char[2]
  numericvalue real
  stringvalue TEXT
  numberlist real[]
  choicelist TEXT[]
  nestednumberlist real[][]    
}

Table parametertypecode {
  parametertypecode char[2] [pk]
  parametertype varchar(16)
  parametertypeexplained TEXT
}

Table modelcovariates {
  modeluuid UUID [pk]
  covariates TEXT[]
  deltabeginbands TEXT[]
  deltaendbands TEXT[]
}

Table modelkfoldresult {
    modeluuid UUID [pk]
    rmsetotal real
    r2total real
    rpiqtotal real
    rmsefoldedmean real
    rmsefoldedstd real
    rpiqfoldedmean real
    rpiqfoldedstd real
    maefoldedmean real
    maefoldedstd real
    mapefoldedmean real
    mapefoldedstd real
    medaefoldedmean real
    r2foldedmean real
    r2foldedsts real
    jsonresult TEXT
    plotresult TEXT
}

Table modeltraintestresult {
    modeluuid UUID [pk]
    rmse real
    r2 real
    rpiq real
    jsonresult TEXT
}

Table pubmodel {
  modeluuid UUID [pk]
  nearestband boolean [default: false]
  interpolateband boolean [default: true]
  averagaband boolean [default: false]
  pickledmodel TEXT  
}

Ref: modelsetups.spectralmodel.modeluuid - modelparameters.modeluuid
Ref: modelparameters.parametertypecode - parametertypecode.parametertypecode
Ref: modelsetups.spectralmodel.modeluuid - modelcovariates.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - modelkfoldresult.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - modeltraintestresult.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - pubmodel.modeluuid
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
        "schema": "modelimplements",
        "table": "modelparameters",
        "command": [
          "modeluuid UUID",
          "hyperparameter varchar(32)",
          "parametertypecode char[2]",
          "numericvalue real",
          "stringvalue TEXT",
          "numberlist real[]",
          "choicelist TEXT[]",
          "nestednumberlist real[][]",
          "PRIMARY KEY (modeluuid, hyperparameter)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelimplements",
        "table": "parametertypecode",
        "command": [
          "parametertypecode char[2]",
          "parametertype varchar(16)",
          "parametertypeexplained TEXT",
          "PRIMARY KEY (parametertypecode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelimplements",
        "table": "modelcovariates",
        "command": [
          "modeluuid UUID",
          "covariates TEXT[]",
          "deltabeginbands TEXT[]",
          "deltaendbands TEXT[]",
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
        "schema": "modelimplements",
        "table": "modelkfoldresult",
        "command": [
          "modeluuid UUID",
          "rmsetotal real",
          "r2total real",
          "rpiqtotal real",
          "rmsefoldedmean real",
          "rmsefoldedstd real",
          "rpiqfoldedmean real",
          "rpiqfoldedstd real",
          "maefoldedmean real",
          "maefoldedstd real",
          "mapefoldedmean real",
          "mapefoldedstd real",
          "medaefoldedmean real",
          "r2foldedmean real",
          "r2foldedsts real",
          "jsonresult TEXT",
          "plotresult TEXT",
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
        "schema": "modelimplements",
        "table": "modeltraintestresult",
        "command": [
          "modeluuid UUID",
          "rmse real",
          "r2 real",
          "rpiq real",
          "jsonresult TEXT",
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
        "schema": "modelimplements",
        "table": "pubmodel",
        "command": [
          "modeluuid UUID",
          "nearestband boolean DEFAULT False",
          "interpolateband boolean DEFAULT True",
          "averagaband boolean DEFAULT False",
          "pickledmodel TEXT",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```

### Preparation codes

TO BE ADDED

```
// i = integer; r = real; s = string;
// il = integerlist; rl = realnumberlist; sl = stringlist
// in = integeternestdlist, rn = real number nested list
```
