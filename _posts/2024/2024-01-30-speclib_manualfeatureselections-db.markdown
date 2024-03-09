---
layout: post
title: Schema manualfeatureselections
categories: libspec-db
excerpt: "Design and setup for schema manualfeatureselections, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-30 11:27'
modified: '2024-01-30 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

When formulating a Machine Learning (ML) model in the xSpectre spectral processing environment, the user can define a manual feature selection. A manual feature selection overrides all other feature selection option and is intended to be used for final model definition before deployment.

This post contains the general design of the schema **manualfeatureselections** for the spectre library and processing system postgreSQL database.

## Purpose of the manualfeatureselections schema

The purpose of the **manualfeatureselections** schema is to define the covariates selected for modeling a target feature. The idea is to interatively identify a parsimonious set of robust covariates using the ML tools for feature selection and then deploy the model with a manual feature selection.

Manual feature selection is applied after any corrections, data transformation,  averaging, harmonizating etc (as defined in the schemas **spectraprepsteps** and **spectrafitsteps**). The schema only contains a single table, _manualfeatures_.

### Illustration of the modelsetups schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-manualfeatureselections.png">
<img src="../../images/DBML_schema-manualfeatureselections.png"></a>
<figcaption>Manualfeatureselections DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra manualfeatureselection library'
}

Table modelsetups.spectralmodel {
  modeluuid  char(36)
  morecolumns char(1)
}

Table manualfeatures {
  modeluuid  char(36) [pk]
  covariates TEXT[]
  deltabeginbands TEXT[]
  deltaendbands TEXT[]
}

Ref: modelsetups.spectralmodel.modeluuid - manualfeatures.modeluuid
```
## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the model manualfeatureselections schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "manualfeatureselections",
        "table": "manualfeatures",
        "command": [
          "modeluuid UUID",
          "covariates TEXT[]",
          "deltabeginbands TEXT[]",
          "deltaendbands TEXT[]",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```
