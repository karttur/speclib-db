---
layout: post
title: Setup schema
categories: libspec-db
excerpt: "Setup schema"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-07 11:27'
modified: '2024-01-07 T18:17:25.000Z'
comments: true
share: true

---

## Introduktion

## xspeclib code

```
{
  "process": [
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sensors"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "samples"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelsteps"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "modelimplements"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectraprepsteps"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrafitsteps"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "splicecorrections"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "generalfeatureselections"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "manualfeatureselections"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "targetfeatureselections"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "featureagglomerations"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "featureagglomerations"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "hyperparamtunings"
      }
    },
    {
      "processid": "createschema",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "plots"
      }
    }
  ]
}

```
