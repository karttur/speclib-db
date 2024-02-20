---
layout: post
title: Samples schema
categories: libspec-db
excerpt: "Design and setup for schema samples, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-15 11:27'
modified: '2024-01-15 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

Each sample (specimen) for which the spectral (or probe) scan is to be saved to the database must be defined. For the registration some attributes are required and some are optional. The number of obligatory attributes depends on the _campaign_ to which the sample belongs. Campaigns that are defined as _timeseries_, _geographic_ and _profiles_ require extra attributes.

This post contains the general design of the schema **samples** for the xspectre spectral library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the spectrometers schema

The purpose of the **samples** schema is to keep track of all specimens that are scanned, whether the scanning is done with a spectral sensor or a probe. All samples that are saved in the database must be linked to a campaign. That link will also define the user (owner), type of sensor, muzzle and probe(s) used in the scanning. But because each campaign can use any instance of the particular models defined also the uuids of the individual sensor, muzzle and probe(s) is saved with each sample. Then of course every sample is given a name and a title. The generic definition of a sample then uses four taxonomic levels: family, species, brand and version. These levels can be interpreted with flexibility as long as they are consistently applied within the same campaign.

If the campaign is a timeseries the ordinal range of the sample is automatically added; if the campaign is geographic the longitude and latitude must be added; if the campaign is a profile, the relative profile position (e.g. depth) must be given.

To improve the visualisation when exploring and analysing the spectra and probe properties, and any properties estimated from these data, each specimen can be given a customised symbolisation to be used in graphical presentations.

To store samples in the databases and retain the outlined information, the schema **samples** contains 5 tables. The table public.sampleifourl is a list that links  extended and external information to a sample. The remaining 4 tables include:

- public.sample (sample metadata and campaign to which sample belongs)
- public.samplesymbol (the symbol to use fore representing the sample graphically)
- public.samplelocation (geographic position and profile depth)
- public.symbols (definition of all available symbols)

### Illustration of the samples schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-samples.png">
<img src="../../images/DBML_schema-samples.png"></a>
<figcaption>Samples DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for sample library'
}

Table campaign.campaign {
  campaignuuid char(36) [pk]
  morecolumns char(1)
}

Table sample {
 sampleuuid char(36)
 sampledatetime timestamp [pk]
 campaignuuid char(36) [pk]
 samplename varchar(32) [pk]
 sampleabbr varchar(16)
 samplelabel varchar(32)
 family varchar(32)
 species varchar(32)
 brand varchar(32)
 version varchar(16)
}

Table samplelocation {
 sampleuuid char(36) [pk]
 longitude double
 latitude double
 mindepth float
 maxdepth float
}

Table sampleinfourl {
 sampleuuid char(36) [pk]
 info varchar (255)
 url TEXT
}

Table samplesymbol {
 sampleuuid char(36) [pk]
 samplesymbolid varchar(36)
 label varchar(32)
 color varchar(16)
 size smallint
}

Table symbols {
 samplesymbolid varchar(36) [pk]
 defaultsymbol varchar(32)
 customsymbol TEXT
 symbolfile TEXT
}

Ref: campaign.campaign.campaignuuid < sample.campaignuuid
Ref: sample.sampleuuid - sampleinfourl.sampleuuid
Ref: sample.sampleuuid - samplelocation.sampleuuid
Ref: sample.sampleuuid - samplesymbol.sampleuuid
Ref: samplesymbol.samplesymbolid - symbols.samplesymbolid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the samples schema.


```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sample",
        "table": "sample",
        "command": [
          "sampleuuid char(36)",
          "sampledatetime timestamp",
          "campaignuuid char(36)",
          "samplename varchar(32)",
          "sampleabbr varchar(16)",
          "samplelabel varchar(32)",
          "family varchar(32)",
          "species varchar(32)",
          "brand varchar(32)",
          "version varchar(16)",
          "PRIMARY KEY (campaignuuid,samplename,sampledatetime)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sample",
        "table": "samplelocation",
        "command": [
          "sampleuuid char(36)",
          "longitude double",
          "latitude double",
          "mindepth float",
          "maxdepth float",
          "PRIMARY KEY (sampleuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sample",
        "table": "sampleinfourl",
        "command": [
          "sampleuuid char(36)",
          "info varchar(255)",
          "url TEXT",
          "PRIMARY KEY (sampleuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "sample",
        "table": "samplesymbol",
        "command": [
          "sampleuuid char(36)",
          "samplesymbolid varchar(36)",
          "label varchar(32)",
          "color varchar(16)",
          "size smallint",
          "PRIMARY KEY (sampleuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "samples",
        "table": "symbols",
        "command": [
          "samplesymbolid varchar(36)",
          "defaultsymbol varchar(32)",
          "customsymbol TEXT",
          "symbolfile TEXT",
          "PRIMARY KEY (samplesymbolid)"
        ]
      }
    }
  ]
}
```
