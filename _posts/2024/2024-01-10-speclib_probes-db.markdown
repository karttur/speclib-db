---
layout: post
title: Schema probes
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

In the terminology used with the xSpectre range of spectrometers, the word _sensor_ is reserved for spectral sensing devices. Other devices, for example Ion Selective Electrodes (ISEs), that are put in direct contact with a sample (whether the sample is a soil, liquid or gas) are called _probes_. To keep these two types of instruments separated they reside in two different schemas in the xSpectre postgreSQL database; [**sensors**](../speclib_sensors-db) and **probes**.

This post contains the general design of the schema **probes** for the xspectre spectral library and processing system. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the probes schema

The purpose of the **probes** schema is to allow defining any external probe that can be carried by the xSpectre [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html). At time of writing this post in February 2024, the following kinds of probes have been tested with the  [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html):

- Ion Selective Electrodes (e.g. for H<sup>+</sup>, NO<sub>3</sub><sup>2-</sup>, NH<sub>4</sub><sup>+</sup>, Ca<sup>2+</sup>),
- Soil penetrometers (e.g. for temperature, humidity, sailinity, pH and NPK),
- Combustable gases (MQ4 sensors for e.g. CH<sub>4</sub>),
- Microphone,
- Soil humidity (via resistivity or capacitivity), and
- Pressure (e.g. for soil compactness).

Most of these probes return a single signal to the [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) microcontroller, but some return more complex data (e.g. using the I<sup>2</sup>C or ModBus protocols). To accommodate the flexibility of allowing definition of the different kinds of probes and the various return signals, the probes schema is divided into 5 separate table. Two of the tables are supporting lists:

- public.probeinfourl (extended information and url links for each probe type and model), and
- public.proberegister (defining the measured physical quantities and units of each probe instrument).

The remaining 3 tables include:

- public.probemodel (containing each probe type and model that can be used with the xspectrolum<b>+</b>),
- public.probe (itemises each individual copy of _probemodel_), and
- public.probecalibration (calibration for each individual probe and the physical quantities measured by that particular probe).

As probes do not have any way to be individually identified (i.e. they do not have any internal memory) they are given the same uuid as the sensor they are first used together with. This in effect means that any calibration is stored along with the sensor (not the probe itself) and users that employ probes must take care to use the same probe with the same sensor to retain accurate observations.

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

Table probemodel {
  probeid varchar(36) [primary key]
  source varchar(32)
  product varchar(32)
  model varchar(32)
  dn_max smallint [NOT NULL]
  samplestate varchar(8)
  substance varchar(16)
  lab_or_field varchar(1)
  status varchar(1)
}

Table probeinfourl {
 probeid varchar(36) [primary key]
 info TEXT
 url TEXT
}

Table support.samplestate {
  samplestatecode char(1) [primary key]
  samplestate varchar(32)
}

Table proberegister {
  probeid varchar(36) [pk]
  quantity varchar(32) [pk]
  unit varchar(32)
  registerkey varchar(16)
}

Table probe {
  probeid varchar(36)
  probeuuid UUID [primary key]
  sensoruuid UUID
  serialnr varchar(16)
}

Table probecalibration {
  probeuuid UUID [pk]
  quantity varchar(32) [pk]
  gain real
  off_set real
}

Table sensors.sensor {
  sensoruuid UUID [primary key]
  morecolumns char(1)
}

Ref: probemodel.samplestate - support.samplestate.samplestatecode
Ref: sensors.sensor.sensoruuid < probe.sensoruuid
Ref: probemodel.probeid - probeinfourl.probeid
Ref: probemodel.probeid < proberegister.probeid
Ref: probemodel.probeid < probe.probeid
Ref: probe.probeuuid < probecalibration.probeuuid
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
          "model varchar(32)",
          "dn_max smallint NOT NULL",
          "samplestate varchar(8)",
          "substance varchar(16)",
          "lab_or_field varchar(1)",
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
          "info TEXT",
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
        "table": "probe",
        "command": [
          "probeid varchar(36)",
          "probeuuid UUID",
          "sensoruuid UUID",
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
          "probeuuid UUID",
          "quantity varchar(32)",
          "gain real",
          "off_set real",
          "PRIMARY KEY (probeuuid,quantity)"
        ]
      }
    }
  ]
}
```
