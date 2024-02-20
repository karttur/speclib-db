---
layout: post
title: Campaign schema
categories: libspec-db
excerpt: "Design and setup for schema campaigns, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-13 11:27'
modified: '2024-01-13 T18:17:25.000Z'
comments: true
share: true

---

## Introduction

Observations obtained with xSpectre´s [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) spectrometer are generally, but not obligatory, associated with a _campaign_. A _campaign_ usually focuses on a particular substance, like soil, fabric, paper, wine, grain; almost any substance can be the focus of a campaign.

A campaign is bound to a specific combination of a sensor model and a muzzle model. "Campaigns" for comparing the performance of different sensors and muzzles can be defined, but are more of development projects and not ordinary "campaigns".

This post contains the general design of the schema **campaigns** for the xspectre spectral library postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the campaigns schema

The purpose of the campaigns schema is to define specific observation tasks centering on a defined substance. A campaign is usually started for building a spectral database for a particular task - like determining the quality of Swedish soils. A campaign is bound to a specific combination of a sensor model (_spectrometermodel_) and a muzzle model (_muzzlemodel_). The campaign can, however apply two, or more, setups (spectromuzzles), as long as they have identical sensorid and muzzleid (but varying (uuids). The actual sensor+muzzle(+probe) used for scanning a sample is registered with each scan; in the [**scan** schema](../speclib_campaign_scan-db).

Apart from the sensorid and muzzleid, _campaign_ attributes include owner, title, label, substance under study etc. A campaign can also have the following boolean dimensions:

- timeseries,
- geographic, and
- profile.

A campaign can include probes. Any number of different probes can be added and be used in conjunction with the spectral scanning. Either for complementing the spectral data or as a means for creating the reference data to predict with the spectral data.

To define campaigns with different set-ups, the **campaigns** schema is divided into 5 tables. Two of the tables
 are supporting lists:

- public.campaigninfourl (extended information and url links for each campaign)
- public.campaigngeoregion (predefined geographic region names)

The remaining 3 tables include:

- public.campaign (general campaign attribute data),
- public.campaignsensor (definition of the campaign sensor and muzzle combination),
- public.campaignprobes (list of probe models applied).

### Further development

Campaigns need to be possible to organise hierarchically (e.g. parent campaign = red wine; child campaign = Spanish red wine; grand child campaign = Rioja red wines).

Campaigns should also have extended status regarding model readiness, ownership, contribution etc, and for large campaigns with many users there should be the possibilities with using different sensor models and muzzle models (but with the same spectrometer method and overlapping spectral ranges).

### Illustration of the campaigns schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-campaign.png">
<img src="../../images/DBML_schema-campaign.png"></a>
<figcaption>Campaign DBML database structure</figcaption>
</figure>

### DBML Code


```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for campaign library'
}

Table campaign {
 useruuid char(36) [pk]
 campaignid varchar(32) [pk]
 campaignuuid char(36)
 campaigntitle varchar(64)
 campaignlabel varchar(32)
 version varchar(8)
 substance varchar(24)
 scanstate varchar(8)
 timeseries boolean
 geographic boolean
 profile boolean
 createdatetime timestamp
 status varchar(1)
}

Table campaignsensor {
  campaignuuid varchar(36) [pk]
  sensorid varchar(36)
  muzzleid varchar(36)
}

Table campaignprobes {
 campaignuuid varchar(36) [pk]
 probeid varchar(36) [pk]
 required boolean
}

Table campaigngeoregion {
 campaignuuid char(36) [pk]
 georegion varchar(64)
}

Table campaigninfourl {
 campaignuuid char(36) [pk]
 info varchar (255)
 url TEXT
}

Table users.users {
  useruuid char(36) [pk]
  morecolumns char(1)
}

Table sensors.sensormodels {
  sensorid varchar(36) [pk]
  morecolumns char(1)
}

Table muzzles.muzzlemodels {
  muzzleid varchar(36) [pk]
  morecolumns char(1)
}

Table probes.probemodels {
  probeid varchar(36) [pk]
  morecolumns char(1)
}

Ref: campaign.campaignuuid - campaigngeoregion.campaignuuid
Ref: campaign.campaignuuid - campaigninfourl.campaignuuid
Ref: campaign.useruuid - users.users.useruuid
Ref: campaign.campaignuuid - campaignsensor.campaignuuid
Ref: campaign.campaignuuid < campaignprobes.campaignuuid
Ref: campaignsensor.sensorid - sensors.sensormodels.sensorid
Ref: campaignsensor.muzzleid - muzzles.muzzlemodels.muzzleid
Ref: campaignprobes.probeid - probes.probemodels.probeid
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the campaigns schema.


```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "campaign",
        "command": [
          "useruuid char(36)",
          "campaignid varchar(32)",
          "campaignuuid char(36)",
          "campaigntitle varchar(64)",
          "campaignlabel varchar(32)",
          "version varchar(8)",
          "substance varchar(24)",
          "scanstate varchar(8)",
          "timeseries boolean",
          "geographic boolean",
          "profile boolean",
          "createdatetime timestamp",
          "status varchar(1)",
          "PRIMARY KEY (useruuid, campaignid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "campaignsensor",
        "command": [
          "campaignuuid varchar(36)",
          "sensorid varchar(36)",
          "muzzleid varchar(36)",
          "PRIMARY KEY (campaignuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "campaignprobes",
        "command": [
          "campaignuuid varchar(36)",
          "probeid varchar(36)",
          "required boolean",
          "PRIMARY KEY (campaignuuid)"
        ]
      }
    },

    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "spectrascansettings",
        "command": [
          "campaignuuid varchar(36)",
          "prescans",
          "warmup",
          "etc???"
          "PRIMARY KEY (campaignuuid)"
        ]
      }
    },


    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "campaigngeoregion",
        "command": [
          "campaignuuid varchar(36)",
          "georegion varchar(36)",
          "PRIMARY KEY (campaignuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "campaigns",
        "table": "campaigninfourl",
        "command": [
          "campaignuuid varchar(36)",
          "info varchar(255)",
          "url TEXT",
          "PRIMARY KEY (campaignuuid)"
        ]
      }
    }
  ]
}
```
