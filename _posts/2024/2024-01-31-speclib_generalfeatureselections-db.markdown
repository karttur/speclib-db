---
layout: post
title: Schema generalfeatureselections
categories: libspec-db
excerpt: "Design and setup for schema scan, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-31 11:27'
modified: '2024-01-31 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

Campaign data is usually compoed of a large number of samples, each with hundreds or even thousands of recorded wavelength reflectances. The data can also contain errors (outliers) and using all the data can lead to over-parameterised models. It is also inevitable that some wavelengths (or bands) contain less information compared to others; a band with no variation (i.e. constant reflection) does not contain any relevant information for modelling.

The tables in the schema **generalfeatureselections** are only useful when formulating a Machine Learning (ML) model. For independent predictions using only selected features you need to manually define the covariates identified from the general (and/or specific) feature selections.

This post contains the general design of the schema **generalfeatureselections** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the general feature selection schema

General data cleaning and selection methods analyse the independent features (the covariates) disregarding both the target and the regressor used for estimation of the target variations. The methods are more crude compared to the specific methods that compare the covariates in relation to the target and the estimator. The general methods are, however comparatively fast and applying the general methods means that all subsequent processing in the process-flow will use the cleaned and reduced dataset and thus also become faster.

The present version of the process-flow implement three generic methods for general feature selection/reduction:

- outlier detection and removal,
- variance threshold feature selection, and
- ward feature clustering.

### Outlier detection and removal

To remove outliers the process-flow implements four different outlier detectors available in the package Python scikit learn:

- [IsolationForest (IF)](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html),
- [EllipticEnvelope (EE)](https://scikit-learn.org/stable/modules/generated/sklearn.covariance.EllipticEnvelope.html),
- [LocalOutlierFactor (LOF)](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.LocalOutlierFactor.html), and
- [OneClassSVM (OCS)](https://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html).

### Variance threshold feature selection

Variance threshold is a method that removes all low-variance features with variances below a stated threshold. The removal is unrelated to the target features and the regressor. To neutralise the range of the input features the data should be rescaled. The parameters to be set is the scaler and the threshold for retaining or discarding a feature.

### ward feature clustering

he Ward clustering implemented in the process-flow has the advantage that you can tune the number of clusters to request from the main agglomeration.

### Illustration of the generalfeatureselections schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-generalfeatureselections.png">
<img src="../../images/DBML_schema-generalfeatureselections.png"></a>
<figcaption>generalfeatureselections DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for general feature selection, scikit learn model'
}

Table modelsetups.spectralmodel {
  modeluuid  char(36)
  morecolumns char(1)
}

Table generalfeatureselection {
  modeluuid UUID [pk]
  removeoutliers boolean
  variancethreshold boolean
  wardclustering boolean
  agglomerativeclustering boolean
}

Table removeoutliers {
  modeluuid UUID [pk]
  detectorcode varchar(3)
  contamination real
}

Table variancethreshold {
  modeluuid UUID [pk]
  scalercode varchar(4)
  threshold real
}

Table wardclustering {
  modeluuid UUID [pk]
  nclusters smallint
  affinity varchar(16)
  tunewardclustering boolean  
}

Table tunewardclustering {
  modeluuid UUID [pk]
  kfolds smallint [default: 3]
  cluster smallint[]  
}

Table detectorcode {
  detectorcode varchar(3) [pk]
  detector varchar(32)
  scikitlearn TEXT
}

Table scalercode {
  scalercode varchar(4) [pk]
  scaler varchar(32)
  scikitlearn TEXT
}

Ref: modelsetups.spectralmodel.modeluuid - generalfeatureselection.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - removeoutliers.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - variancethreshold.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - wardclustering.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - tunewardclustering.modeluuid
Ref: removeoutliers.detectorcode - detectorcode.detectorcode
Ref: variancethreshold.scalercode - scalercode.scalercode
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
        "schema": "generalfeatureselections",
        "table": "globalfeatureselection",
        "command": [
          "modeluuid char(36)",
          "removeoutliers boolean)",
          "variancethreshold boolean",
          "wardclustering boolean",
          "agglomerativeclustering boolean",
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
        "schema": "generalfeatureselections",
        "table": "removeoutliers",
        "command": [
          "modeluuid char(36)",
          "detectorcode varchar(3)",
          "contamination real",
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
        "schema": "generalfeatureselections",
        "table": "variancethreshold",
        "command": [
          "modeluuid char(36)",
          "scalercode varchar(4)",
          "threshold real",
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
        "schema": "generalfeatureselections",
        "table": "wardclustering",
        "command": [
          "modeluuid char(36)",
          "nclusters smallint",
          "affinity varchar(16)",
          "tunewardclustering boolean",
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
        "schema": "generalfeatureselections",
        "table": "tunewardclustering",
        "command": [
          "modeluuid char(36)",
          "kfolds smallint DEFAULT 3",
          "cluster smallint[]",
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
        "schema": "generalfeatureselections",
        "table": "detectorcode",
        "command": [
          "detectorcode varchar(3)",
          "detector varchar(32)",
          "scikitlearn TEXT",
          "PRIMARY KEY (detectorcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "generalfeatureselections",
        "table": "scalercode",
        "command": [
          "scalercode varchar(4)",
          "signalmean real[]",
          "scaler varchar(32)",
          "scikitlearn TEXT"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
