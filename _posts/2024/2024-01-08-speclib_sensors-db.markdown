---
layout: post
title: Schema sensors
categories: libspec-db
excerpt: "Design and setup for schema sensors, xspectre postgreSQL database"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-08 11:27'
modified: '2024-01-08 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

In the terminology used with the xSpectre range of spectrometers, the word _sensor_ is reserved for spectral sensing devices. Other devices, for example Ion Selective Electrodes (ISEs), that are put in direct contact with a sample (whether the sample is a soil, liquid or gas) are called _probes_. To keep these two classes of instruments separated they reside in two different schemas in the xSpectre postgreSQL database; **sensors** and [**probes**](../speclib_probes-db).

This post contains the general design of the schema **sensors** for the xspectre spectral library and processing system. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the sensors schema

The purpose of the **sensors** schema is to allow defining any spectral sensor that can be carried by the xSpectre [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html). At time of writing this post in February 2024, [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) can carry any of the following spectral sensors:

- [Hamamatsu C12880MA (288 band VIS grating sensor)](https://www.hamamatsu.com/jp/en/product/type/C12880MA/index.html)
- [Hamamatsu C14384MA-01 (192 band NIR grating sensors)](https://www.hamamatsu.com/eu/en/product/optical-sensors/spectrometers/mini-spectrometer/C14384MA-01.html)
- [Hamamatus C14273/C13272-03/C14273 (MEMS-FPI NIR sensor)](https://www.hamamatsu.com/content/dam/hamamatsu-photonics/sites/documents/99_SALES_LIBRARY/ssd/c14273_kacc1265e.pdf)
- [AMS OSRAM AS7262 6 bands VIS filter sensor](https://ams.com/as7262)
- [AMS OSRAM AS7263 6 bands NIR filter sensor](https://ams.com/as7263)
- [AMS OSRAM AS7341 (11 bands filter sensor)](https://ams.com/AS7341)
- [AMS OSRAM AS7343 (14 bands filter sensor)](https://ams.com/AS7343)
- [AMS OSRAM AS7421 (64-bands filter sensor)](https://ams.com/as7421)

To accommodate the flexibility of allowing definition of all of the above, the **sensors** schema is divided into 7 separate table. Two of the tables are supporting lists:

- public.technology (list of technologies used for separating the light signal in the spectral sensors), and
- public.sensorinfourl (extended information and url links for each sensor model).

The table _support.spectrum_ belongs to the **support** schema and is a list of abbreviation/codes used for defining overall wavelength ranges.

The remaining 5 tables include:

- public.sensormodel (containing each sensor model used in the xspectrolum<b>+</b>)
- public.sensors (itemises each individual copy of the _sensormodel_ as mounted in actual spectrometers),
- public.gratingsensors (individual characteristics of each mounted grating sensor),
- public.hamamatuscalibration (calibration parameters for the Hamamatsu 5-degree polynomial used for calibration of each individual Hamamatsu grating and MEMS-FPI sensor)
- public.filtersensor (individual band wavelength [wl] center and full width at half maximum [fwhm] for filter based sensors).

The key used for linking spectral sensors to a spectrometer is the field _sensoruuid_, where uuid is an abbreviation for _Universally Unique Identifier_. The _sensoruuid_ is automatically generated with each sensor registration, The same uuid is also given to the spectrometer carrying the sensor and registered in the memory of the microprocessor of that sensor.

### Illustration of the sensors schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-sensors.png">
<img src="../../images/DBML_schema-sensors.png"></a>
<figcaption>Sensors DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectral sensors'
}

Table sensormodel {
  sensorid varchar(36) [primary key]
  source varchar(32)
  product varchar(32)
  model varchar(32)
  dn_max smallint [NOT NULL]
  bands smallint [NOT NULL]
  spectrum varchar(16)
  technology varchar(16)
  status char(1)
}

Table sensorinfourl {
 sensorid varchar(36) [primary key]
 info TEXT
 url TEXT
}

Table sensor {
  sensorid varchar(36)
  sensoruuid UUID [primary key]
  serialnr varchar(16)
}

Table filtersensor [headercolor: #3498DB] {
  sensoruuid UUID [primary key]
  wl real[] [NOT NULL]
  fwhm real[] [NOT NULL]
}

Table gratingsensor {
  sensoruuid UUID [primary key]
  beginpixel smallint
  endpixel smallint
  maxpixel smallint
  minwl real
  maxwl real
  fwhm real
}

Table hamamatsucalibration {
  sensoruuid UUID [primary key]
  a0 double
  b1 double
  b2 double
  b3 double
  b4 double
  b5 double
}

Table technology {
  technology varchar(16) [primary key]
  info TEXT
}

Table support.spectrum {
  spectrum varchar(16) [primary key]
  info TEXT
}

Ref: sensormodel.sensorid - sensorinfourl.sensorid
Ref: sensor.sensoruuid - filtersensor.sensoruuid
Ref: sensor.sensoruuid - gratingsensor.sensoruuid
Ref: gratingsensor.sensoruuid - hamamatsucalibration.sensoruuid
Ref: sensor.sensorid > sensormodel.sensorid
Ref: sensormodel.technology - technology.technology
Ref: sensormodel.spectrum - support.spectrum.spectrum
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the sensors schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sensors",
        "table": "sensormodel",
        "command": [
          "sensorid varchar(36)",
          "source varchar(32)",
          "product varchar(32)",
          "model varchar(32)",
          "dn_max smallint NOT NULL",
          "bands smallint NOT NULL",
          "spectrum varchar(16)",
          "technology varchar(16)",
          "status char(1)",
          "PRIMARY KEY (sensorid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sensors",
        "table": "sensorinfourl",
        "command": [
          "sensorid varchar(36)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (sensorid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sensors",
        "table": "sensor",
        "command": [
          "sensorid varchar(36)",
          "sensoruuid UUID",
          "serialnr varchar(16)",
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
        "schema": "sensors",
        "table": "filtersensor",
        "command": [
          "sensoruuid UUID",
          "wl real[]",
          "fwhm real[]",
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
        "schema": "sensors",
        "table": "gratingsensor",
        "command": [
          "sensoruuid UUID",
          "beginpixel smallint",
          "endpixel smallint",
          "maxpixel smallint",
          "minwl real[]",
          "maxwl real[]",
          "fwhm real[]",
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
        "schema": "sensors",
        "table": "hamamatsucalibration",
        "command": [
          "sensoruuid UUID",
          "a0 double precision DEFAULT 0",
          "b1 double precision DEFAULT 0",
          "b2 double precision DEFAULT 0",
          "b3 double precision DEFAULT 0",
          "b4 double precision DEFAULT 0",
          "b5 double precision DEFAULT 0",
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
        "schema": "sensors",
        "table": "technology",
        "command": [
          "technology varchar(16)",
          "info TEXT",
          "PRIMARY KEY (technology)"
        ]
      }
    }
  ]
}
```
