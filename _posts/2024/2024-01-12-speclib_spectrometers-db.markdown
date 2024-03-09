---
layout: post
title: Schema spectrometers
categories: libspec-db
excerpt: "Design and setup for schema spectrometers, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-12 11:27'
modified: '2024-01-12 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

In the xSpectre terminology a spectrometer is a composition consisting of a spectral sensor (defined in the schema [sensors](../speclib_sensors-db/)) and one or more muzzles, carrying the lamp and sample holder (defined in the schema [muzzles](../speclib_muzzles-db)). Physically the spectral sensor is attached to the xspeclum PCB that is mounted in the spectrometer body while the muzzle carries the smaller xspecled PCB. The spectrometer body and muzzle are joined with a bayonet mount.

Spectrometers can also attach external probes (defined in the schema [probes](../speclib_probes-db)). The attachment is either through a more general GX16 (aviation) port or, for Ion Selective Electrodes, a BNC (coaxial) port. The probes as such are not defined as part of the spectrometer. The availability of the GX16 and BNC ports, however, are defined with each spectrometer model.

This post contains the general design of the schema **spectrometers** for the xspectre spectral library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the spectrometers schema

The purpose of the **spectrometers** schema is to make it possible to represent any combination of sensors (spectrometer bodies or xspeclum PCBs) and muzzles (or xspecled PCBs). The schema is also used for defining which models come with external ports (ie. GX16 and/or BNC).

All realised combinations of
- sensor,
- PCB version,
- GX16 port, and
- BNC port.

must be registered as a spectrometer model in the table _spectrometermodel_ and each actual copy of a model registered in the table _spectrometer_. As each _spectrometer_ contains a uniquely identified _sensor_, the universally unique identifiner (uuid) of a spectrometer is set to the sensoruuid of the attached sensor. Each spectrometer can be linked to any number of muzzles (via the _muzzleuuid)_, with each unique combination registered separately in the table _spectromuzzle_ and given a unique _spectromuzzleuuid_.

Because both individual sensors (xspeclum PCB/spectrometer body) and individual light sources (xspecled PCB/spectrometer muzzle) vary slightly in function and performance, each combination must be individually calibrated. The calibration consists of three parts:

1. power supply (voltage) that determines the electromagnetic emission from the lamp(s),
2. integration time tuning, and
3. reference spectra (depends on the spectral method but must always be there).

To accommodate the flexibility of allowing definition of all of the above, the **spectrometers** schema is divided into 7 separate table. One of the tables, _public.spectrometerinfourl_ is a support table. The remaining 6 tables include:

- public.spectrometermodel (definition of all realised spectrometer models),
- public.spectrometer (itemised copies of spectrometer models),
- public.spectromuzzle (itemised combinations of spectrometers and muzzles),
- public.spectromuzzlepower (calibrated voltage power for individual spectrometer+muzzle combinations),
- public.spectromuzzlereftuning (tuning of integration times across the sensor spectral range for individual spectrometer+muzzle combinations), and
- public.spectromuzzlerefspectra (reference spectra for individual spectrometer+muzzle combinations).

### Illustration of the muzzles schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-spectrometers.png">
<img src="../../images/DBML_schema-spectrometers.png"></a>
<figcaption>Spectrometers DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectrometers'
}

Table sensors.sensor {
  sensorid varchar(36)
  sensoruuid UUID [primary key]
  serialnr varchar(16)
}

Table muzzles.muzzle {
  muzzleuuid UUID [primary key]
  muzzleid varchar(36)
  serialnr varchar(16)
}

Table spectrometermodel {
 sensorid varchar(36)
 source varchar(32)
 product varchar(32)
 model varchar(32)
 version varchar(8)
 pcb varchar(16)
 gx16port varchar(1)
 bncport varchar(1)
 spectrometermodelid varchar(36) [pk]
 status char(1)
}

Table spectrometerinfourl {
 spectrometermodelid varchar(36) [primary key]
 info TEXT
 url TEXT
}

Table spectrometer {
 sensoruuid UUID [pk]
 spectrometermodelid varchar(36)
 createdate date
 status char(1)
}

Table spectromuzzle {
 sensoruuid UUID [pk]
 muzzleuuid UUID [pk]
 spectromuzzleuuid UUID
 createdatetime timestamp
}

Table spectromuzzlepower {
 spectromuzzleuuid UUID [pk]
 mv integer
 createdatetime timestamp [pk]
}

Table spectromuzzlereftuning {
 spectromuzzleuuid UUID [pk]
 ms_array smallint[]
 beginpixelarray smallint[]
 endpixelarray smallint[]
 createdatetime timestamp [pk]
}

Table spectromuzzlerefspectra {
 spectromuzzleuuid UUID [pk]
 createdatetime timestamp [pk]
 signalmean real[]
 signalstd real[]
 darkmean real[]
 darkstd real[]
 tag varchar(16) [default: 'default',pk]
}

Ref: sensors.sensor.sensorid - spectrometermodel.sensorid
Ref: spectrometermodel.spectrometermodelid - spectrometerinfourl.spectrometermodelid
Ref: spectrometermodel.spectrometermodelid < spectrometer.spectrometermodelid
Ref: sensors.sensor.sensoruuid - spectrometer.sensoruuid
Ref: spectrometer.sensoruuid - spectromuzzle.sensoruuid
Ref: spectromuzzle.muzzleuuid < muzzles.muzzle.muzzleuuid
Ref: spectromuzzle.spectromuzzleuuid - spectromuzzlepower.spectromuzzleuuid
Ref: spectromuzzle.spectromuzzleuuid - spectromuzzlereftuning.spectromuzzleuuid
Ref: spectromuzzle.spectromuzzleuuid - spectromuzzlerefspectra.spectromuzzleuuid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the spectrometers schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectrometermodel",
        "command": [
          "sensorid varchar(36)",
          "source varchar(32)",
          "product varchar(32)",
          "model varchar(32)",
          "version varchar(8)",
          "pcb varchar(16)",
          "gx16port varchar(1)",
          "bncport varchar(1)",
          "spectrometermodelid varchar(36)",
          "status char(1)",
          "PRIMARY KEY (spectrometermodelid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectrometerinfourl",
        "command": [
          "spectrometermodelid varchar(36)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (spectrometermodelid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectrometer",
        "command": [
          "sensoruuid UUID",
          "spectrometermodelid varchar(36)",
          "createdate date",
          "status char(1)",
          "PRIMARY KEY (sensoruuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectromuzzle",
        "command": [
          "sensoruuid UUID",
          "muzzleuuid UUID",
          "spectromuzzleuuid UUID",
          "createdatetime timestamp",
          "PRIMARY KEY (sensoruuid,muzzleuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectromuzzlepower",
        "command": [
          "spectromuzzleuuid UUID",
          "mv smallint",
          "createdatetime timestamp",
          "PRIMARY KEY (spectromuzzleuuid,createdatetime)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectromuzzlereftuning",
        "command": [
          "spectromuzzleuuid UUID",
          "ms_array smallint[]",
          "beginpixelarray smallint[]",
          "endpixelarray smallint[]",
          "createdatetime timestamp",
          "PRIMARY KEY (spectromuzzleuuid,createdatetime)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "spectrometers",
        "table": "spectromuzzlerefspectra",
        "command": [
          "spectromuzzleuuid UUID",
          "createdatetime timestamp",
          "signalmean real[]",
          "signalstd real[]",
          "darkmean real[]",
          "darkstd real[]",
          "tag varchar(16)",
          "PRIMARY KEY (spectromuzzleuuid,createdatetime)"
        ]
      }
    }
  ]
}
```
