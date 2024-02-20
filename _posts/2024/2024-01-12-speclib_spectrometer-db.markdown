---
layout: post
title: Spectrometers schema
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

In the xSpectre terminology a spectrometer is a composition consisting of a spectral sensor (defined in the schema [sensors](../speclib_sensor-db/)) and one or more muzzles, carrying the lamp and sample holder (defined in the schema [muzzles](../speclib_muzzle-db)). Physically the spectral sensor is attached to the xspeclum PCB that is mounted in the spectrometer body while the muzzle carries the smaller xspecled PCB. The spectrometer body and muzzle are joined with a bayonet mount. A spectrometer body can be linked to any number and type of xSpectre muzzle.

Spectrometers can also attach external probes (defined in the schema [probes](../speclib_probes-db)). The attachment is either through a more general GX16 (aviation) port or, for Ion Selective Electrodes, a BNC (coaxial) port. The probes as such are not defined as part of the spectrometer. The availability of the GX16 and BNC ports, however, are defined with each spectrometer model.

This post contains the general design of the schema **spectrometers** for the xspectre spectral library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the spectrometers schema

The purpose of the **spectrometers** schema is to make it possible to represent any combination of sensors (spectrometer bodies or xspeclum PCBs) and muzzles (or xspecled PCBs).

All realised combinations of
- sensor,
- PCB version,
- GX16 port, and
- BNC port

must be registered as a spectrometer model in the table _spectrometermodels_ and each actual copy of a model registered in the table _spectrometers_. As each _spectrometer_ contains a uniquely identified sensor, the universally unique identifiner (uuid) of a spectrometer is set to the sensoruuid of the attached sensor. Each spectrometer can be linked to any number of muzzles (via the _muzzleuuid)_, with each unique combination registeted separately in the table _spectromuzzles_ and given a unique _spectromuzzleuuid_.

Because both individual sensors (xspeclum PCB/spectrometer body) and individual light sources (xspecled PCB/spectrometer muzzle) vary slightly in shape, function and performance, each combination must be individually calibrated. The calibration consists of three parts:

1. power supply (voltage) that determines the electromagnetic emission from the lamp(s),
2. integration time tuning, and
3. reference spectra (depends on the spectral method but must always be there).

To accommodate the flexibility of allowing definition of all of the above, the spectrometers schema is divided into 7 separate table. One of the tables, public.spectrometerinfo is a support table. The remaining 6 tables include:

- public.spectrometermodels (definition of all realised spectrometer models),
- public.spectrometers (itemised copies of spectrometer models),
- public.spectromuzzles (itemised combinations of spectrometers and muzzles),
- public.spectromuzzlepower (calibrated voltage power for individual spectrometer+muzzle),
- public.spectromuzzlereftuning (tuning of integration times across the sensor spectral range for individual spectrometer+muzzle), and
- public.spectromuzzlerefspectra (reference spectra for individual spectrometer+muzzle).

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

Table sensors.sensors {
  sensorid varchar(36)
  sensoruuid char(36) [primary key]
  serialnr varchar(16)
}

Table muzzles.muzzles {
  muzzleuuid char(36) [primary key]
  muzzleid varchar(36)
  serialnr varchar(16)
}

Table spectrometermodels {
 sensorid varchar(36)
 source varchar(32)
 product varchar(32)
 model varchar(24)
 version varchar(8)
 pcb varchar(16)
 gx16port varchar(1)
 bncport varchar(1)
 spectrometermodelid varchar(36) [pk]
 status char(1)
}

Table spectrometerinfourl {
 spectrometermodelid varchar(36) [primary key]
 info varchar (255)
 url TEXT
}

Table spectrometers {
  sensorid varchar(36)
 sensoruuid char(36) [pk]
 spectrometermodelid varchar(36)
 serialnr varchar(16)
 gx16port varchar(1)
 bncport varchar(1)
 createdate date
 status char(1)
}

Table spectromuzzles {
 sensoruuid char(36) [pk]
 muzzleuuid char(36) [pk]
 spectromuzzleuuid char(36)
 createdatetime timestamp
}

Table spectromuzzlepower {
 spectromuzzleuuid char(36) [pk]
 millivoltage integer
 createdatetime timestamp [pk]
}

Table spectromuzzlereftuning {
 spectromuzzleuuid char(36) [pk]
 msecarray smallint[]
 startpixelarray smallint[]
 endpixelarray smallint[]
 createdatetime timestamp [pk]
}

Table spectromuzzlerefspectra {
 spectromuzzleuuid char(36) [pk]
 createdatetime timestamp [pk]
 signalmean real[]
 signalstd real[]
 darkmean real[]
 darkstd real[]
 tag varchar(16) [default: 'default',pk]
}

//Ref: spectrometer.sensoruuid - sensors.sensors.sensoruuid
 Ref: sensors.sensors.sensorid - spectrometermodels.sensorid
Ref: spectrometermodels.spectrometermodelid - spectrometerinfourl.spectrometermodelid
Ref: spectrometermodels.spectrometermodelid < spectrometers.spectrometermodelid
Ref: sensors.sensors.sensoruuid - spectrometers.sensoruuid
Ref: spectrometers.sensoruuid - spectromuzzles.sensoruuid
Ref: spectromuzzles.muzzleuuid < muzzles.muzzles.muzzleuuid
Ref: spectromuzzles.spectromuzzleuuid - spectromuzzlepower.spectromuzzleuuid
Ref: spectromuzzles.spectromuzzleuuid - spectromuzzlereftuning.spectromuzzleuuid
Ref: spectromuzzles.spectromuzzleuuid - spectromuzzlerefspectra.spectromuzzleuuid
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
        "schema": "spectrometer",
        "table": "spectrometermodels",
        "command": [
          "sensorid varchar(36)",
          "source varchar(32)",
          "product varchar(32)",
          "model varchar(24)",
          "version varchar(8)",
          "pcb varchar(16)",
          "gx16port varchar(1)",
          "bncport varchar(1)",
          "spectrometermodelid varchar(36)",
          "serialnr varchar(16)",
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
          "info varchar (255)",
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
        "table": "spectrometers",
        "command": [
          "sensorid varchar(36)",
          "sensoruuid char(36)",
          "spectrometermodelid varchar(36)",
          "serialnr varchar(16)",
          "gx16port varchar(1)",
          "bncport varchar(1)",
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
        "schema": "spectrometer",
        "table": "spectromuzzles",
        "command": [
          "sensoruuid char(36)",
          "muzzleuuid char(36)",
          "spectromuzzleuuid char(36)",
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
        "schema": "spectrometer",
        "table": "spectromuzzlepower",
        "command": [
          "spectromuzzleuuid char(36)",
          "millivoltage smallint",
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
        "schema": "xspectrometer",
        "table": "spectromuzzlereftuning",
        "command": [
          "spectromuzzleuuid char(36)",
          "msecarray smallint[]",
          "startpixelarray smallint[]",
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
        "schema": "xspectrometer",
        "table": "spectromuzzlerefspectra",
        "command": [
          "spectromuzzleuuid char(36)",
          "createdatetime timestamp",
          "signalmean real[]",
          "signalstd real[]",
          "darkmean real[]",
          "darkstd real[]",
          "tag varchar()16",
          "PRIMARY KEY (spectromuzzleuuid,createdatetime)"
        ]
      }
    }
  ]
}
```
