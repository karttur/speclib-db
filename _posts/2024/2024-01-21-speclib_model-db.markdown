---
layout: post
title: Model schema
categories: libspec-db
excerpt: "Design and setup for schema model, xspectre postgreSQL spectral library"
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

The **model** schema functions like a root for the definition of Machine Learning (ML) modelling and predicting of properties from spectral data. The **model** schema itself contains tables for
- metadata defining a model,
- process steps to include for
 setting up model testing and validation,
- model results, and
- validated models.

Tables defining the covariate correction and selection, parameter settings for model testing and validation, plotting and hyperparameter tuning are defined in separate schemas (see below).

This post contains the general design of the schema **model** for the xspectra library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the model schema

The purpose of the **model** schema is to
 1. designate a target feature, name, title and campaign for a machine learning (ML) model based on spectral data as covariates,
 2. to define the framework for creating a spectral ML model for predicting physico-chemical properties for a target feature,
 3. record the parameterisation and performance of individual models, and
 4. publish validated models.

The tables of the schema include:

- spectralmodel (target feature, name, title, campaign etc),
- modelinfourl (extended information and url links for each model),
- spectraprepsteps (boolean options for preparatory steps for correcting, transforming and extracting spectral data),
- spectrafitsteps (boolean options for harmonisation, correction and information enhancement of prepared spectral data),
- featureselectsteps (boolean options for ML feature selection and [further] dimension reduction),
- modelarguments (hyperparameter settings for best fitted models, reporting of feature importance and model validation),
- parametertypecode (final model list of hyperparameters and settings, and plots),
- modelcovariates (final model selected covariates),
- modelkfoldresults (model statistical performance and plots for best fitted models),
- modeltraintestresults (model statistical performance and plots for best fitted models), and
- publishedmodel (models published for independent prediction).

Each of the optional boolean processes listed in the tables:
- spectraprepsteps,
- spectrafitsteps, and
- featureselectsteps.

are specified in separate tables that then link to a set of other schemas and tables.

### spectraprepsteps

The specta preparatory steps (table: _spectraprepsteps_) include:

- datatransform,
- scattercorrection,
- meancentring,
- movingaverage,
- movingaverageclusters, and
- averageclusters.

The processes behind these options are all optional and applied to the orignal spectral data (as recorded in the schema [**scan**](../speclib-scan-db). If several of the processes listed above are applied, they are performed in a pipe-line sequence. Of the latter three (movingaverage, movingaverageclusters and averageclusters), only one can be applied in each model.

The processes are detailed in the schema [**preprocessing**](../speclib_model-preprocessing-db/).

### spectrafitsteps

The specta fitting steps (table: _spectrafitsteps_) include:

- bandharmonisation,
- splicecorrection,
- derivatives,
- standardise,
- pca, and
- manualfeatureselection.

These processes can be applied either to the orignal spectral data as recorded in the schema [**scan**](../speclib-scan-db) or to the outcome of the specta preparatory steps.

_bandharmonisation_ is a required step for harmonising spectra from different spectometers (models or individual copies) for use in a shared model (whether for model formualtion or prediction).

_splicecorrection_ is either for shifting the spectral signal from one type of spectrometer model domain (sensor+muzzle) to another model domain, or for correcting for a secondary feature (besides the target feature) that is directly observed (by other than specral means) in each sample (e.g. moisture content, pH, salt content etc). The complexity of the splice correction has rendered it to have its own schema - **splicecorrection**.

Any _derivatives_ (change between two consecutive wavelengths) are calulated after all the band related adjustments are done. _derivates_ is a boolean parameter that if set to _true_ generates derivatives alongside the band data.

Also _standardise_ is a boolean parameter, but if set to true the gain and offset values to convert the spectra (derivates) to a standardised range must be retained. Thus the standardisation is saved in a dedicated table under the **spectrafit** schema.

Principal component analysis (_pca_) is a data compression and information enhancement algorithm. For model formulation and validataion, pca is automatically retrieved from the data after the pre-processing and fitting steps.

### featureselectsteps

The pre-processing and fitting steps above can both increase and decrease the number of covariates. If requested, a manual selection of a subset among the surviving covariates can be set. if a manual selection is autorised (by setting the boolean variable _manualfeatureselection_ to _true_), all other feature selections options are forced to _false_. This would typically be done for a final model formulation that is both effective and parsimonious.

If _manualfeatureselection_ is set to _false_, one or more of the remaining, automatic, feature selection options can be called for model calibration and validation, inlcuding:

- global feature selection (_globalfeatureselection_, applied withour considering the target feature or the regressor)
- target feature selection (_targetfeatureselection_, applied considering the target feature)
- feature agglomeration (_featureagglomeration_, applied considering the target feature), and
- model feature selection (_modelfeatureselection_, applied considering both the target feature and the regressor).

For each of these options the actual process and its parameterisation must be set. These expansions of feature selection are defined in 4 different schemas named as indicated in the list above.

### Illustration of the model schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema_model.png">
<img src="../../images/DBML_schema_model.png"></a>
<figcaption>Model DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra model library'
}

Table spectralmodel {
  campaignuuid char(36) [pk]
  subsampleselectcode char(3) [default:'avg']
  targetfeatureuuid char(36) [pk]
  targetfeatureminvalue real [default:0]
  targetfeaturemaxvalue real [default:0]
  targetfeaturelist TEXT[]
  regressor varchar(16) [pk]
  edition varchar(16) [pk]
  version char(2) [pk]
  publicitycode char(4)
  createadatetime timestamp
  modeluuid  char(36)
}

Table subsampleselect {
  subsampleselectcode char[3]
  subsampleselect char[32]
}
// min;max;avg;std;rng;1st,2nd,3rd,4th

Table modelinfourl {
  modeluuid  char(36) [pk]
  modelname varchar(32)
  modeltitle varchar(128)
  modeldescript TEXT
  modelrestriction TEXT
  modeldisclaimer TEXT
  info varchar(255)
  url TEXT  
}

Table spectraprepsteps {
  modeluuid char(36) [pk]
  datatransformcode char(2)
  scattercorrection boolean
  meancentring boolean
  movingaverage boolean
  movingaverageclusters boolean
  averageclusters boolean
}

Table spectrafitsteps{
  modeluuid char(36) [pk]
  bandharmonisation boolean
  splicecorrection boolean
  derivatives boolean
  standardise boolean
  pca boolean
}

Table featureselectsteps {
  modeluuid char(36) [pk]
  manualfeatureselection boolean
  globalfeatureselection boolean
  targetfeatureselection boolean
  featureagglomeration boolean
  modelfeatureselection boolean
}

Table modelarguments {
  modeluuid char(36) [pk]
  hyperparameter varchar(32) [pk]
  parametertypecode char[2]
  numericvalue real
  stringvalue TEXT
  numberlist real[]
  choicelist TEXT[]
  nestednumberlist real[][]  
  hyperparametertuning boolean
  featureimportance boolean
  modelvalidation boolean   
}

Table parametertypecode {
  parametertypecode char[2] [pk]
  parametertype varchar(16)
  parametertypeexplained TEXT
}

// i = integer; r = real; s = string;
// il = integerlist; rl = realnumberlist; sl = stringlist
// in = integeternestdlist, rn = real number nested list

Table modelcovariates {
  modeluuid  char(36) [pk]
  spectralbands smallint[]
  derivatestartbands smallint[]
  derivateendbands smallint[]
}

Table modelkfoldresults {
    modeluuid  char(36) [pk]
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
    jsonresults TEXT
    plotresults TEXT
    pickledmodel TEXT
}

Table modeltraintestresults {
    modeluuid  char(36) [pk]
    rmse real
    r2 real
    rpiq real
    jsonresults TEXT
    pickledmodel TEXT
}

Table publishedmodel {
  modeluuid char(36) [pk]
  nearestband boolean
  interpolateband boolean
  averagaband boolean
  spectralbands smallint[]
  derivatestartbands smallint[]
  derivateendbands smallint[]
  pickledmodel TEXT  
}
Ref: spectralmodel.modeluuid - modelinfourl.modeluuid
Ref: spectralmodel.subsampleselectcode - subsampleselect.subsampleselectcode
Ref: spectralmodel.modeluuid - spectraprepsteps.modeluuid
Ref: spectralmodel.modeluuid - spectrafitsteps.modeluuid
Ref: spectralmodel.modeluuid - featureselectsteps.modeluuid
Ref: spectralmodel.modeluuid - modelarguments.modeluuid
Ref: modelarguments.parametertypecode - parametertypecode.parametertypecode
Ref: spectralmodel.modeluuid - modelcovariates.modeluuid
Ref: spectralmodel.modeluuid - modelkfoldresults.modeluuid
Ref: spectralmodel.modeluuid - modeltraintestresults.modeluuid
Ref: spectralmodel.modeluuid - publishedmodel.modeluuid
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
          "sampleuuid char(36)",
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
          "info varchar (255)",
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
          "info varchar (255)",
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
          "sampleuuid char(36)",
          "subsample char(2)",
          "scanuuid char(36)",
          "spectromuzzleuuid char(36)",
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
          "scanuuid char(36)",
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
          "scanuuid char(36)",
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
          "scanuuid char(36)",
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
          "scanuuid char(36)",
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
          "sampleuuid char(36)",
          "subsample varchar(8)",
          "probeuuid char(36)",
          "prepcode char(2)",
          "mode varchar(16)",
          "scanuuid char(36)",
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
          "scanuuid char(36)",
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
          "info varchar (255)",
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
          "info varchar (255)",
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
