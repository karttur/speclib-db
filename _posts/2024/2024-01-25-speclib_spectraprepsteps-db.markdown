---
layout: post
title: Spectraprepsteps schema
categories: libspec-db
excerpt: "Design and setup for schema  spectraprepsteps, xspectre postgreSQL spectral library"
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

# SCATTERCORRECTON?

## Introduction

The xSpectre processing chain for correcting and harmonising spectral data is divided in two parts:
- spectra preparation processes (**spectraprepsteps**), applied equally to all spectra regardless of range and resolution, both in model formulation and model prediction, and
- spectra fitting processes ([**spectrafitsteps**](../speclib_spectrafitsteps-db/)), that require parameters defined during model formulation when used for model predictions.

This post contains the general design of the schema **spectraprepsteps** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the spectraprepsteps schema

The purpose of the **spectraprepsteps** schema is to store the combination of generic pre-processing steps defined for spectral data analysis and modelling. I.e. the same steps and parameters are applied both in model formulation and for model predictions. The processes, with one exceptions (the Yeo-Johnson power transformation) do not require any parameterisation surviving from model formulation to model prediction. The principal generic preparation methods include:

- data transformation (_datatransfromcode_),
- moving average (_movingaverage_),
- moving average clusters (_movingaveragecluster_),
- average clusters (_averageclusters_), and
- band ranging (_bandranging_).

The methods can be used in sequence, but only one of the three averaging/clustering processes (_movingaverage_, _movingaverageclusters_ and _averageclusters_) can be applied in any particular model.

### Data transformation

Data transformations implemented in the xspectre modelling environment include:

- logarithmic transformation,
- exponential transformation,
- square root transformation,
- inversion, and
- power (Yeo-Johnson) transformation.

Of the above transformations, the exponential transformation and Yeo-Johnson power transformation require parameters when shifting form model formulation to model predicition.

### Scatter correction

There are 2 proper methods for scatter correction and 4 simple statistical methods:

- Multiplicative Scatter Correction (MSC),
- Standard Normal Variate (SNV),
- Average (AVG),
- Median (MED),
- Minimum (MIN), and
- Maximum (MAX).

Scatter correction is only an option if the scans to be used contain multiple subsambles of the same sample.

### Moving average

Moving average applies a single moving average function from a defined range with a single central start band and a single central end band. The filter kernel can either be defined as the spectral width to average, or the number of destination bands to retrieve. If the destination spectra must have precisely equally separated band centres this is set as a boolean argument. The same result can also be achieved using the _averageclusters_ process below.

Note that for camapaings employing more than one single isntrument (spectrometer + muzzle) it is highly recommended to apply a harmonization to a pre-defiend spectral range and resoluton alreade at the model formulation stage.

### Moving average clusters

Operates like a cluster of moving averages, where each cluster is retrieved from sections of the full source spectra using individually defined moving average settings as defined in the [movingaverage](#moving-average) section above.

### Average clusters

The _averageclusters_ process clusters source bands to a list of predefined central bands, each with an individual set kernel (window). A global decay function can be set to define the influence of the spectral proximity within the kernels. This can be used for e.g. simulating the full width at half maximum (FWHM) distribution of wavelengths influencing broad band spectral sensors.

### Band ranging

Extracting or excluding sections of the full band range of any spectral sensor. Band ranging can be used for solving several issues, including removing spectral regions with uncertain signals, harmonising between different spectral sensors or for simulating the response of sensors with more restricted ranges.

Band ranging is done after any moving average to avoid edge effects.

### Illustration of the spectraprepsteps schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema_spectraprepsteps.png">
<img src="../../images/DBML_schema_spectraprepsteps.png"></a>
<figcaption>Spectraprepsteps DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectraprepsteps library'
}

Table modelsetups.spectralmodel {
  modeluuid  char(36)
  morecolumns char(1)
}

Table datatransformcode {
  datatransformcode char(2) [NOT NULL, pk]
  datatransform varchar(32) [NOT NULL]
  scikitlearncommand TEXT
}

Table datatransform {
  modeluuid UUID [pk]
  datatransformcode char(2) [NOT NULL]
  exponent real
  lambda real
}

Table movingaverage {
  modeluuid UUID [pk]
  spectralwidth real [default:0]
  destinationbands smallint [default:0]
  beginband smallint [NOT NULL]
  endband smallint [NOT NULL]
}

Table movingaverageclusters {
  modeluuid UUID [pk]
  spectralwidths real[] [NOT NULL]
  destinationbands smallint[] [NOT NULL]
  beginbands smallint[] [NOT NULL]
  endbands smallint[] [NOT NULL]
}

Table averageclusters {
  modeluuid UUID [pk]
  centralbands smallint[] [NOT NULL]
  kernels real[] [NOT NULL]
  decayfunction real[] [NOT NULL]
}

Table bandranging {
  modeluuid UUID [pk]
  beginbands smallint[] [NOT NULL]
  endbands smallint[] [NOT NULL]
}

Ref: modelsetups.spectralmodel.modeluuid - datatransform.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - movingaverage.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - movingaverageclusters.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - averageclusters.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - bandranging.modeluuid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the model spectraprepsteps schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectraprepsteps",
        "table": "datatransformcode",
        "command": [
          "datatransformcode char(2)",
          "datatransform varchar(32)",
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
        "schema": "spectraprepsteps",
        "table": "datatransform",
        "command": [
          "modeluuid UUID",
          "datatransformcode char(2)",
          "exponent real",
          "lambda real",
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
        "schema": "spectraprepsteps",
        "table": "scattercorrection",
        "command": [
          "modeluuid UUID",
          "MSC boolean",
          "SNV boolean",
          "double boolean",
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
        "schema": "spectraprepsteps",
        "table": "movingaverage",
        "command": [
          "modeluuid UUID",
          "spectralwidth real DEFAULT 0",
          "destinationbands smallint DEFAULT 0",
          "beginband smallint NOT NULL",
          "endband smallint NOT NULL",
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
        "schema": "spectraprepsteps",
        "table": "movingaverageclusters",
        "command": [
          "modeluuid UUID",
          "spectralwidth real[] NOT NULL",
          "destinationbands smallint[] NOT NULL",
          "beginband smallint[] NOT NULL",
          "endband smallint[] NOT NULL",
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
        "schema": "spectraprepsteps",
        "table": "averageclusters",
        "command": [
          "modeluuid UUID",
          "centralbands smallint[] NOT NULL",
          "kernels real[] NOT NULL",
          "decayfunction real[] NOT NULL",
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
        "schema": "spectraprepsteps",
        "table": "bandranging",
        "command": [
          "modeluuid UUID",
          "beginbands smallint[] NOT NULL",
          "endbands smallint[] NOT NULL",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED

## Resources

[Multiplicative Scatter Correction (MSC)](https://guifh.github.io/RNIR/MSC.html)

[Two scatter correction techniques for NIR spectroscopy in Python](https://nirpyresearch.com/two-scatter-correction-techniques-nir-spectroscopy-python/)

[Procedures for Wavelength Calibration and Spectral Response Correction of CCD Array Spectrometers](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4651606/)

Excellent Youtube lecture on [Preprocessing 1. Centering & Scaling](https://www.youtube.com/watch?v=hOZfAFIQ4sg)
