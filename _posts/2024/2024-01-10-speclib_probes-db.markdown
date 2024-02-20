---
layout: post
title: Setup schema probes
categories: libspec-db
excerpt: "Setup schema probes"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-10 11:27'
modified: '2024-01-10 T18:17:25.000Z'
comments: true
share: true

---

## Introduktion

In the terminology used with the xSpectre range of spectrometers, the word _sensor_ is reserved for spectral sensing devices. Other devices, for example Ion Selective Electrodes (ISEs), that are put in direct contact with a sample (whether the sample is a soil, liquid or gas) are labelled _probes_. To keep these two types of instruments separated they reside in two different schemas in the xSpectre postgreSQL database; **sensors** and **probes**.

This post contains the general design of the schema **probes** for the xspectre spectral library. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the probes schema

The purpose of the probes schema is to allow defining any external probe that can be carried by the xSpectre [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html). At time of writing this post in February 2024, the following kinds of probes have been tested with the  [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html):

- Ion Selective Electrodes (e.g. for H, NO3, NH4, Ca)
- Soil penetrometers (e.g. for temperature, humidity, sailinity, pH and NPK)
- Combustable gases (MQ4 sensors for e.g. CH4)
- Microphone
- Soil humidity (via resistivity or capacitivity)
- Pressure (e.g. for soil compactness)

Most of these probes returns a single signal to the [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) microcontroller, but some return more complex data (e.g. using the I2C or ModBus protocols). To accommodate the flexibility of allowing definition of the different kinds of probes and the various return signals, the probes schema is divided into 5 separate table. Two of the tables are supporting lists:

- public.probeinfourl (extended information and url links for each probe type and model), and
- public.proberegisters (defining the measured physical quantities and units of each probe instrument)

The remaining 3 tables include:

- public.probemodels (containing each probe type and model that can be used with the xspectrolum<b>+</b>)
- public.probes (itemizes each individual copy of the probemodels), and
- public.probecalibration (calibration for each individual probe and physical quantities measured by that particular probe)

As probes do not have any way to be individually identified (i.e. they do not have any internal memory) they are given the same uuid as the sensor they are used together with. This in effect means that any calibration is stored along with the sensor (not the probe itself) and users that employ probes must take care to use the same probe with the same sensor to retain accurate observations.

### Illustration of the probes schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-probes.png">
<img src="../../images/DBML_schema-probes.png"></a>
<figcaption>Probes DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for probes'
}

Table probemodels {
  probeid varchar(36) [primary key]
  source varchar(32)
  product varchar(32)
  model varchar(24)
  maxdn smallint [NOT NULL]
  physicalstate varchar(8)
  substance varchar(16)
  labfielduse varchar(1)
  status varchar(1)
}

Table probeinfourl {
 probeid varchar(36) [primary key]
 info varchar (255)
 url TEXT
}

Table proberegisters {
  probeid varchar(36) [pk]
  quantity varchar(32) [pk]
  unit varchar(32)
  registerkey varchar(16)
}

Table probes {
  probeid varchar(36)
  probeuuid char(36) [primary key]
  sensoruuid char(36)
  serialnr varchar(16)
}

Table probecalibration {
  probeuuid char(36) [pk]
  quantity varchar(32) [pk]
  gain real
  offset real
}

Table sensors.sensors {
  sensoruuid char(36) [primary key]
  morecolumns char(1)
}

Ref: sensors.sensors.sensoruuid < probes.sensoruuid
Ref: probemodels.probeid - probeinfourl.probeid
Ref: probemodels.probeid < proberegisters.probeid
Ref: probemodels.probeid < probes.probeid
Ref: probes.probeuuid < probecalibration.probeuuid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the probes schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes",
        "table": "probemodels",
        "command": [
          "probeid varchar(36)",
          "source varchar(32)",
          "product varchar(32)",
          "model varchar(24)",
          "maxdn smallint [NOT NULL]",
          "physicalstate varchar(8)",
          "substance varchar(16)",
          "labfielduse varchar(1)",
          "status varchar(1)",
          "PRIMARY KEY (probeid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes",
        "table": "probeinfourl",
        "command": [
          "probeid varchar(36)",
          "info varchar (255)",
          "url TEXT",
          "PRIMARY KEY (probeid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes",
        "table": "proberegisters",
        "command": [
          "probeid varchar(36)",
          "quantity varchar(32)",
          "unit varchar(32)",
          "registerkey varchar(16)",
          "PRIMARY KEY (probeid, quantity)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes",
        "table": "probes",
        "command": [
          "probeid varchar(36)",
          "probeuuid char(36)",
          "sensoruuid char(36)",
          "serialnr varchar(16)",
          "PRIMARY KEY (probeuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "probes",
        "table": "probecalibration",
        "command": [
          "probeuuid char(36)",
          "quantity varchar(32)",
          "gain real",
          "offset real",
          "PRIMARY KEY (probeuuid,quantity)"
        ]
      }
    }
  ]
}
```
