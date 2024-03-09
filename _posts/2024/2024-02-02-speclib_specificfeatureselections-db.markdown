---
layout: post
title: Schema specificfeatureselections
categories: libspec-db-todo
excerpt: "Design and setup for schema specificfeatureselections, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-02-02 11:27'
modified: '2024-02-02 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

To select the features (covariates) that relate to a specific target (e.g. soil properties) the process-flow implements univariate feature selection. The most advanced options for reducing the number of covariates consider both the target property and the applied regressor. The process-flow includes two methods that can be used for this kind of model related feature selection: Permutation Importance Selection and Recursive Feature Elimination (RFE).

The parameteisations when applying one of these methods are defined in the **specificfeatureselections** schema.

In the process-flow you can in principle invoke two of the methods, or even all three, but in normal cases you would only use one for each model building exercise.

This post contains the general design of the schema **specificfeatureselections** for the xspectre library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the specific feature selection schema

The **specificfeatureselections** schema defines Machine Learning (ML) methods for selecting the subset of covariates (spectra and spectra derivatives) that most strongly infuenceses the prediction of the variations in a target feature. The methods support constructing parsimonious ML models and thus also avoiding overfitting.

The present version of the process-flow implements one method selecting covariates related to the target feature:
- [SelectKBest](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.SelectKBest.html),

and two methods for selecting covariates related to the both the target feature and the regressor:
- [permutation feature importance](https://scikit-learn.org/stable/modules/permutation_importance.html), and
- [Recurcive Feature Elimination (RFE)](https://scikit-learn.org/stable/modules/feature_selection.html#recursive-feature-elimination).

### Illustration of the specificfeatureselections schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-specificfeatureselections.png">
<img src="../../images/DBML_schema-specificfeatureselections.png"></a>
<figcaption>specificfeatureselections DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for target feature selection, scikit learn model'
}

Table modelsetups.spectralmodel {
  modeluuid UUID
  morecolumns char(1)
}

Table specificfeatureselection {
  modeluuid UUID [pk]
  univariatefeatureselection boolean
  permutationselector boolean
  rfe boolean
}

Table univariatefeatureselection {
  modeluuid UUID [pk]
  selectkbest smallint [default:0]
  selectpercentile smallint [default:0]
  genericunivariateselect boolean
}

Table genericunivariateselect {
  modeluuid UUID [pk]
  hyperparameter varchar(32) [pk]
  parametervalue varchar(32) [pk]
}

Table permutationselector {
  modeluuid UUID [pk]
  permutationrepeats smallint
  n_featurestoselect smallint
  step smallint  
}

Table RFE {
  modeluuid UUID [pk]
  CV boolean
  n_featurestoselect smallint
}

Ref: modelsetups.spectralmodel.modeluuid - specificfeatureselection.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - univariatefeatureselection.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - genericunivariateselect.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - permutationselector.modeluuid
Ref: modelsetups.spectralmodel.modeluuid - RFE.modeluuid
```



## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the specificfeatureselections schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "specificfeatureselections",
        "table": "specificfeatureselection",
        "command": [
          "modeluuid UUID",
          "univariatefeatureselection boolean",
          "permutationselector boolean",
          "rfe boolean",
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
        "schema": "specificfeatureselections",
        "table": "univariatefeatureselection",
        "command": [
          "modeluuid UUID",
          "selectkbest smallint DEFAULT: 0",
          "selectpercentile smallint DEFAULT 0",
          "genericunivariateselect boolean",
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
        "schema": "specificfeatureselections",
        "table": "genericunivariateselect",
        "command": [
          "modeluuid UUID",
          "hyperparameter varchar(32)",
          "parametervalue varchar(32)",
          "PRIMARY KEY (modeluuid, hyperparameter, parametervalue)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "specificfeatureselections",
        "table": "permutationselector",
        "command": [
          "modeluuid UUID",
          "permutationrepeats smallint",
          "n_featurestoselect smallint",
          "step smallint",
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
        "schema": "specificfeatureselections",
        "table": "RFE",
        "command": [
          "modeluuid UUID",
          "CV boolean",
          "n_featurestoselect smallint",
          "PRIMARY KEY (modeluuid)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
