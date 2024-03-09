---
layout: post
title: Schema modelsetups
categories: libspec-db
excerpt: "Design and setup for schema modelsetups, xspectre postgreSQL spectral library"
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

The **modelsetups** schema functions like a root for the testing, evaluation and definition of Machine Learning (ML) models using spectra and spectra derivates as covariates and aiming to predict any physical, chemical and biological property of the substance (target feature) under stury. The **modelsetups** schema itself contains tables for:
- metadata defining a model, and
- process steps to include for setting up model development, testing and validation.

Tables defining the process parameters and algorithms to apply for e.g. covariate correction and selection, parameter settings for model testing and validation, plotting and hyperparameter tuning are defined in separate schemas (see below). The **modelsetups** schema, however, contains the boolean (yes/no) arguments determining whether or not to include a particular process or algorithm in the process chain.

This post contains the general design of the schema **modelsetups** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the modelsetups schema

The purpose of the **modelsetups** schema is to:
 1. designate a target feature, name, title and campaign for a machine learning (ML) model based on spectral data as covariates, and
 2. to define the overall process chain for developing, calibrating and validating a spectral ML model for predicting properties for a target feature.

The tables of the schema include:

- _spectralmodel_ (defining model target feature, regressor, campaign etc),
- _modelinfourl_ (extended information and url links for each model),
- _modeldesignation_ (model meta data for published models),
- _modelsteps_ (the stages of the model formulation processes)
- _sampleselect_ (alternatives for selecting a subset of the campaigns samples;  methods for aggregating subsamples),
- _spectraprepsteps_ (boolean options for preparatory steps for correcting, transforming and extracting spectral that do not [in general] require parameterisation),
- _spectrafitsteps_ (boolean options for harmonisation, correction and information enhancement of prepared spectral data that require parameterisation),
- _featureselectsteps_ (boolean options for ML feature selection and [further] dimension reduction and information enhancement).

The state of the boolean arguments in table _modelsteps_ determines how far the process chain of model formulation is carried out and whether or not to save data to the database, or just show the results temprorarily. If the argument _deployed_ is set to _true_ the model run is saved to the database and can not be changed. The idea is to use the boolean argument steps in _modelsteps_ for facilitating a stepwise definition of the model setup and formulation. This is done by starting with all arguments set to _false_ and then sequentially open steps and exploring the results through both numerical and graphical reporting. Then iteratively finish one step before continuing with the next.

Each of the optional boolean processes listed in the tables:
- _spectraprepsteps_,
- _spectrafitsteps_, and
- _featureselectsteps_,

are specified in separate tables that then link to a set of other schemas and tables.

### spectraprepsteps

The spectra preparatory steps (table: _spectraprepsteps_) include:

- data transformation (_datatransfromcode_),
- moving average (_movingaverage_),
- moving average clusters (_movingaveragecluster_),
- average clusters (_averageclusters_), and
- band ranging (_bandranging_).

The processes behind these steps are all optional and applied to the original spectral data (as recorded in the schema [**scans**](../speclib-scan-db)). If several of the processes listed above are applied, they are performed in a pipe-line sequence. Of the three alternatives for averaging and clustering (_movingaverage_, _movingaverageclusters_ and _averageclusters_), only one can be applied in each model.

The processes are detailed in the schema [**spectraprepsteps**](../speclib_spectraprepsteps-db/).

### spectrafitsteps

The spectra fitting steps (table: _spectrafitsteps_) include:

- splice correction (_splicecorrection_),
- mean centring (_meancentring_),
- derivatives (_derivatives_),
- standardise (_standardise_), and
- princicpal component analysis (_pca_).

Also these process steps are optional and if requested applied either to the original spectral data as recorded in the schema [**scans**](../speclib-scan-db) or to the outcome of the spectra preparatory steps if one or more of these are included in the process chain.

_splicecorrection_ is either for shifting the spectral signal from one type of spectromuzzle combination (sensor+muzzle) to another combination, or for correcting for a secondary feature (besides the target feature) that is directly observed (by other than spectral means) in each sample (e.g. moisture content, pH, salt content etc). The complexity of the splice correction has rendered it to have its own schema - [**splicecorrection**](../speclb/splice-correction-db/).

_meancentring_ removes the average spectrum from all the spectra in a campaign when formulating a model. This removes any offset and enhances the information content. The average spectrum must be saved with the final model formulation and applied in any subsequent model predictions.

Any _derivatives_ are calculated after all the band related adjustments are done. The parameters for generating derivatives are given in the table _derivatives_. If derivates are used, the option is to either use them with or without including also the original bands. _derivatives_ can also be used for smoothing the spectra (by setting a filter and requesting the 0th derivative while omitting the original data).

Also _standardise_ is a boolean parameter, but if set to true the gain and offset values to convert the spectra (derivates) to a standardised range must be retained and used in any subsequent model prediction for standardising the input spectra in the same manner as done in the deployed model formulation.

Principal component analysis (_pca_) is a data compression and information enhancement algorithm. For model formulation and validation, any pca transformation is applied after the pre-processing and other fitting steps. The eigenvalues for all retained components must be used in any subsequent predictions.

### featureselectsteps

The pre-processing and fitting steps above can both increase and decrease the number of covariates. If requested, a manual selection of a subset among the surviving covariates can be set. If a manual selection is authorised (by setting the boolean variable _manualfeatureselection_ to _true_), all other feature selection options in the table _featureselectsteps_ are forced to _false_. This would typically be done for defining a deployed model formulation that is both effective and parsimonious.

If _manualfeatureselection_ is set to _false_, one or more of the remaining, automatic, feature selection options can be called for model calibration and validation, including:

- general feature selection (_generalfeatureselection_ applied without considering the target feature or the regressor),
- specific feature selection, and (_specificfeatureselection_ applied considering the target feature and/or regressor).

For each of these options the actual process and its parameterisation must be set. These expansions of feature selection are defined in different schemas named as indicated in the list above.

### Illustration of the modelsetups schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema_modelsetups.png">
<img src="../../images/DBML_schema_modelsetups.png"></a>
<figcaption>Modelsetups DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra modelsetups library'
}

Table spectralmodel {
  modelname  varchar(32) [pk]
  campaignuuid UUID [pk]
  targetfeatureuuid UUID [pk]
  regressor varchar(16) [pk]
  edition varchar(16) [pk]
  version char(2) [pk]
  publicitycode char(4)
  createadatetime timestamp
  modeluuid UUID
}

Table modelsteps {
  modeluuid UUID [pk]
  deployd boolean [default: false]
  published boolean [default: false]
  reportprepsteps boolean [default: true]
  reportfitsteps boolean [default: true]
  reportfeatureimportance boolean [default: true]
  reportmodelvalidation boolean [default: true]
}

Table modeldesignation {
  modeluuid UUID [pk]
  modeltitle varchar(128)
  modeldescription TEXT
  modelrestriction TEXT
  modeldisclaimer TEXT
}

Table sampleselect {
  modeluuid UUID [pk]
  targetfeatureminvalue real [default:0]
  targetfeaturemaxvalue real [default:0]
  targetfeaturevaluelist TEXT[]
  sqlsearch TEXT
  scattercorrectioncode char(3) [default: 'avg']
}
// min;max;avg;std;rng;1st,2nd,3rd,4th

Table modelinfourl {
  modeluuid UUID [pk]
  info TEXT
  url TEXT  
}

Table spectraprepsteps {
  modeluuid UUID [pk]
  datatransformcode char(2)
  movingaverage boolean
  movingaverageclusters boolean
  averageclusters boolean
  bandranging boolean
}

Table spectrafitsteps{
  modeluuid UUID [pk]
  splicecorrection boolean
  meancentring boolean
  derivatives boolean
  standardise boolean
  pca boolean
}

Table featureselectsteps {
  modeluuid UUID [pk]
  manualfeatureselection boolean
  generalfeatureselection boolean
  specificfeatureselections boolean
}

Ref: spectralmodel.modeluuid - modelsteps.modeluuid
Ref: spectralmodel.modeluuid - modelinfourl.modeluuid
Ref: spectralmodel.modeluuid - modeldesignation.modeluuid
Ref: spectralmodel.modeluuid - sampleselect.modeluuid
Ref: spectralmodel.modeluuid - spectraprepsteps.modeluuid
Ref: spectralmodel.modeluuid - spectrafitsteps.modeluuid
Ref: spectralmodel.modeluuid - featureselectsteps.modeluuid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the modelsetups schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelsetups",
        "table": "spectralmodel",
        "command": [
          "modelname  varchar(32)",
          "campaignuuid UUID",
          "targetfeatureuuid UUID",
          "regressor varchar(16)",
          "edition varchar(16)",
          "version char(2)",
          "publicitycode char(4)",
          "createadatetime timestamp",
          "modeluuid UUID",
          "PRIMARY KEY (modelname, campaignuuid, targetfeatureuuid, regressor, edition, version)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelsetups",
        "table": "modelsteps",
        "command": [
          "modeluuid UUID",
          "deployd boolean DEFAULT false",
          "published boolean DEFAULT false",
          "reportprepsteps boolean DEFAULT true",
          "reportfitsteps boolean DEFAULT true",
          "reportfeatureimportance boolean DEFAULT true",
          "reportmodelvalidation boolean DEFAULT true",
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
        "schema": "modelsetups",
        "table": "modeldesignation",
        "command": [
          "modeluuid UUID",
          "modeltitle varchar(128)",
          "modeldescription TEXT",
          "modelrestriction TEXT",
          "modeldisclaimer TEXT",
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
        "schema": "modelsetups",
        "table": "sampleselect",
        "command": [
          "modeluuid UUID",
          "targetfeatureminvalue real DEFAULT 0",
          "targetfeaturemaxvalue real DEFAULT 0",
          "targetfeaturevaluelist TEXT[]",
          "sqlsearch TEXT",
          "scattercorrectioncode char(3) DEFAULT 'avg'",
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
        "schema": "modelsetups",
        "table": "modelinfourl",
        "command": [
          "modeluuid UUID",
          "info TEXT",
          "url TEXT",
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
        "schema": "modelsetups",
        "table": "spectraprepsteps",
        "command": [
          "modeluuid UUID",
          "datatransformcode char(2)",
          "movingaverage boolean",
          "movingaverageclusters boolean",
          "averageclusters boolean",
          "bandranging boolean",
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
        "schema": "modelsetups",
        "table": "spectrafitsteps",
        "command": [
          "modeluuid UUID",
          "splicecorrection boolean",
          "meancentring boolean",
          "derivatives boolean",
          "standardise boolean",
          "pca boolean",
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
        "schema": "modelsetups",
        "table": "featureselectsteps",
        "command": [
          "modeluuid UUID",
          "manualfeatureselection boolean",
          "generalfeatureselection boolean",
          "specificfeatureselection boolean",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED

```
// i = integer; r = real; s = string;
// il = integerlist; rl = realnumberlist; sl = stringlist
// in = integeternestdlist, rn = real number nested list
```
