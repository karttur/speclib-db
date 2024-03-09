---
layout: post
title: Spectrafitsteps schema
categories: libspec-db
excerpt: "Design and setup for schema spectrafitsteps, xspectre postgreSQL spectral library"
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

The xSpectre processing chain for correcting and harmonising spectral data is divided in two parts:
- spectra preparation processes ([**spectraprepsteps**](../speclib_spectraprepsteps-db/) applied equally to all spectra regardless of range and resolution, both in model formulation and model prediction)
- spectra fitting (**spectrafitsteps**, requires parameters that relate to default properties when used for model predictions)

This post contains the general design of the schema **spectrafitsteps** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the spectrafitsteps schema

The purpose of the **spectrafitsteps** schema is twofold; during the model formulation (calibration and validation) some parameters must be set for each included step. For some of the steps, the model formulation will result in an expanded set of parameters that will be added to the database once the model formulation is finalised and deployed; the second purpose of the schema is thus to store the validated parameters for all subsequent applications of the model. The principal fitting methods implemented in the process chain include:

- splice correction (_splicecorrection_),
- derviates (_derivatives_),
- mean centring (_meancentring_(),
- standaridsation (_standardise_), and
- principal component analysis (_pca_).

The fitting steps can be used in sequence.

As a side note: there are other methods that could be applied but are not at present implemented. Autoscaling is not implemented as it is noise-sensitive and thus less suitable for use with lower grade spectral sensors. Pareto and poisson scaling are modifications of autoscaling, but with suppression of noise. They are sometimes used for enhancing weak signals, like in Raman spectroscopy. At time of writing this in February 2024 these scaling methods are not implemented.

### Splice correction

Splicing can be applied for correcting drift between individual sensors, for example due to geometric differences in the sensor-array. The differences can exist both between instances of the same sensor brand and model, and between different models.

Splice correction can also be applied for reducing or eliminating the influence of a defined sample attribute that is not the target feature. Examples could be the moisture content of any material (for example soil), or some chemical constituent (pH, salinity). A prerequisite for applying splice correction in these cases is that the attribute to correct for is observed with some other probe.

### Derivatives

_derivates_ are calculated prior to any mean centring. The user can set the order of the derivative to retrieve, including the 0th order (= original data). Derivatives can be calculated using a filter, including Savitzky-Golay filter, spline, Gaussian filter and  Whittaker–Henderson smoothing. The user can also define an arbitrary kernel and filter the data.

### Mean centring

In the model formulation, _meancentring_ calculates an average spectrum for all the spectra belonging to a campaign. For each wavelength the average spectrum signal is then subtracted from each original spectra. This in effect removes any offset in the original spectra and forces an average signal of zero at each wavelength. This has several advantages, and few, if any, drawbacks, especially if you apply a principal component analysis (pca) prior to the ML modelling.

The _meancentring_ of any additional sample(s), after the model formulation, must apply the same average spectrum as used in the model formulation. Thus this average spectrum is saved and can not be updated once the model is deployed.

### Standardise (Z-score normalization)

Standardisation rescales data to a range between 0 and 1. In the processing chain it is (if selected) applied after the above listed processes. The parameters for any particular model standardisation are saved in the table.

https://www.kdnuggets.com/2020/04/data-transformation-standardization-normalization.html

### pca

Principal component analysis (pca) converts the input bands to vectors sequentially capturing the remaining variation in the source band data. When applying pca the user needs to define the number of components to retain. For deployed models, the definition of these components are saved as a unitary matrix.

### Illustration of the spectrafitsteps schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-spectrafitsteps.png">
<img src="../../images/DBML_schema-spectrafitsteps.png"></a>
<figcaption>Spectrafitsteps DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra fitting library'
}

Table modelsetups.spectralmodel {
  modeluuid  char(36)
  morecolumns char(1)
}

Table splicecorrection {
  modeluuid UUID [pk]
}

Table derivatives {
  modeluuid UUID [pk]
  derivorder smallint
  filter varchar(16)
  kernelsize smallint
  kernelweights smallint[]
  keepspectra boolean
}

Table meancentring {
  modeluuid UUID [pk]
  meanspectrum real[]
}

Table standardise {
  modeluuid UUID [pk]
  gain real[]
  off_set real[]
}

Table pca {
  modeluuid UUID [pk]
  n_components smallint
  unitarymatrix real[][]
}

Ref: modelsetups.spectralmodel.modeluuid - splicecorrection.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - derivatives.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - meancentring.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - standardise.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - pca.modeluuid
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
        "schema": "spectrafitsteps",
        "table": "splicecorrection",
        "command": [
          "modeluuid UUID",
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
        "schema": "spectrafitsteps",
        "table": "derivatives",
        "command": [
          "modeluuid UUID",
          "derivorder smallint",
          "filter varchar(16)",
          "kernelsize smallint",
          "kernelweights smallint[]",
          "keepspectra boolean",
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
        "schema": "spectrafitsteps",
        "table": "meancentring",
        "command": [
          "modeluuid UUID",
          "meanspectrum real[]",
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
        "schema": "spectrafitsteps",
        "table": "standardise",
        "command": [
          "modeluuid UUID",
          "gain real[]",
          "off_set real[]",
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
        "schema": "spectrafitsteps",
        "table": "pca",
        "command": [
          "modeluuid UUID",
          "n_components smallint",
          "unitarymatrix real[][]",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
