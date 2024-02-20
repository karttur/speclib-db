---
layout: post
title: Model preprocessing schema
categories: libspec-db
excerpt: "Design and setup for schema model preprocessing, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-25 11:27'
modified: '2024-01-25 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

In the xSpectre database the spectral data pre-processing steps to apply for any particular analysis or modelling are stored in the table **modelpreprocessing**.

This post contains the general design of the schema **modelpreprocessing** for the xspectre scan library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the scan schema

The purpose of the **modelpreprocessing** schema is to store the combination of pre-processing steps defined for all data analysis and modeling. The principal pre-processing methods include:

- datatransform,
- scattercorrection,
- meancentring,
- movingaverage,
- movingaverageclusters,
- averageclusters,
- manualfeatureselection,
- standardise, and
- principal component analysis (pca)

The methods can be used in sequence. If the pre-process _manualfeatureselection_ is selected, other feature selection options in the main model steps will be ignored.

### datatransform

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

To BE ADDED

### movingaverage

Movingaverage applies a single moving average function from a defined range with a single central start band and a single central end band, either defining the spectral width to average or the number of destination bands to retrieve.

### movingaverageclusters

Operates like a cluster of moving averages, where each cluster is retrieved from sections of the full source spectra using indivdually defined moving average settings as defined in the [movingaverage](#movingaverage) section.

### averageclusters

The averageclusters process clusters source bands to a list of predefined central bands, each with an individual set kernels (windows). A global decay function can be set to define the influence of the spectral proximity within the kernels.

### manualfeatureselection

If set, Manual feature selection is applied after any corrections, data transformation or averaging. The manual feature selection is thus always set vis-a-vis  spectral bands and their derivatives. While the pre-processes of standardisation and pca can be applied to data extracted using manual feature selection, all further feature selection processes in he main modelling steps are precluded.

### Standardise

Standardisation rescales data to a range between 0 and 1. In the xspectre processing chain it is (if selected) applied after the above listed pre-processes. The parameters for any particular model standardisation are saved in the table.

### pca

Principal component analysis (pca) converts the input bands to vectors sequntially capturing the remaining variation in the source band data. When applying pca the user needs to define the number of components to retain. For any model, the definition of these components are saved as a unitary matrix.



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

Table datatransformcode {
  datatransformcode char(2) [NOT NULL, pk]
  datatransform varchar(32) [NOT NULL]
  scikitlearncommand TEXT
}

Table datatransform {
  modeluuid char(36) [pk]
  datatransformcode char(2) [NOT NULL]
  lambda real
}

Table scattercorrection {
  modeluuid char(36) [pk]
  MSC boolean
  SNV boolean
}

Table meancentring {
  modeluuid char(36) [pk]
  scikitlearncommand TEXT
}

Table splicercorrection {
  modeluuid char(36) [pk]
  scikitlearncommand TEXT
}

Table movingaverage {
  modeluuid char(36) [pk]
  spectralwidth real [default:0]
  destinationbands smallint [NOT NULL]
  beginband smallint [NOT NULL]
  endband smallint [NOT NULL]
}

Table movingaverageclusters {
  modeluuid char(36) [pk]
  spectralwidths real[] [NOT NULL]
  destinationbands smallint[] [NOT NULL]
  beginbands smallint[] [NOT NULL]
  endbands smallint[] [NOT NULL]
}

Table averageclusters {
  modeluuid char(36) [pk]
  centralbands smallint[] [NOT NULL]
  kernels real[] [NOT NULL]
  decayfunction real[] [NOT NULL]
}

Table manualfeatureselection {
  modeluuid char(36) [pk]
  nearestband boolean [default: true]
  interpolateband boolean [default: false]
  averagaband boolean [default: false]
  spectralbands smallint[] [NOT NULL]
  derivatestartbands smallint[]
  derivateendbands smallint[]
}

Table standardise {
  modeluuid char(36) [pk]
  gain real
  offset real
  scikitlearncommand TEXT
}

Table pca {
  modeluuid char(36) [pk]
  dstcomponent smallint
  unitarymatrix real[][]
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



  ]
}
```

### Prepartion codes

TO BE ADDED
