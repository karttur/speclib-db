---
layout: post
title: Schema scans
categories: libspec-db
excerpt: "Design and setup for schema scan, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-16 11:27'
modified: '2024-01-16 T18:17:25.000Z'
comments: true
share: true
---

## Introduction

In the xSpectre database the actual spectral (or probe) measurements are stored in the schema **scans**. This post contains the general design of the schema **scans** for the spectral library and processing system postgreSQL database. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the scans schema

The purpose of the **scans** schema is to store recordings from a [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) spectrometer linked to a sample and a campaign. Each scan, regardless of the sensor or probe, is registered as a signal mean and signal standard deviation (std).

To keep track of the sample, campaign and individual instrument set-up for each scan, the **scans** schema is linked to uuid's in the schemas **spectrometers**, **probes** and **samples**.

The definition and registration of a sample resides in the **samples** schema, but any _subsampling_ (e.g. repeated measurements of different parts or slices of a sample) or different sample _preparations_ (e.g. drying, sieving, chemical preservation, homogenisation etc) are registered in the **scans** schema. In addition also the _mode_ (arbitrarily defined) can be stated for any additional sample or instrument alteration (by default it is not used and left empty).

The [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) can be used for applying 4 different spectroscopy methods:

- diffuse reflectance,
- transparency,
- fluorescence, and
- Raman spectroscopy.

Which method to apply is defined by the campaign that also defines the sensor, muzzle and probe models to use. In the **scans** database the signals derived from the 4 methods are saved in separate tables. Data from probe instruments are saved in one common table.

To store scans in the databases and retain the outlined information, the schema **scans** contains 12 tables. 4 of the tables are support lists for defining sample preparations and mode:

- public.spectraprep (list linking sample preparation codes for spectral scanning to methods),
- public.spectramode (expansion of mode for spectral scanning if required),
- public.probprep (list linking sample preparation codes for probe scanning to methods), and
- public.probemode  (expansion of mode for probe scanning if required)

The remaining 8 tables include:

- public.samplescan (links to the sample and defines any subsampling [defaulted to '_A', '_B', '_C' ... if done sequentially] and the timestamp of the scan),
- public.scanspectra (links the scan to the spectral sensor+muzzle of the individual instrument, and defines the sample preparation and any mode),
- public.reflectance (mean and std of diffuse reflectance signals),
- public.transmissivity (mean and std of transmissivity signals),
- public.reflectance (mean and std of  fluorescence signals),
- public.raman (mean and std of Raman signals),
- public.scanprobe (links the scan to the probe of the individual instrument, defines the sample preparation and any mode), and
- public.scanrecord (stores the mean and std of the probe register).

### Illustration of the scan schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-scans.png">
<img src="../../images/DBML_schema-scans.png"></a>
<figcaption>Scan DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra scan library'
}

Table samples.sample {
  sampleuuid UUID
  campaignuuid UUID
  morecolumns char(1)
}

Table spectraprep {
  prepcode char(2) [NOT NULL, pk]
  sampleprep varchar(32) [NOT NULL]
  info TEXT
  url TEXT
}

Table spectramode {
  mode varchar(16) [pk]
  info TEXT
  url TEXT
}

Table spectrometers.spectromuzzle {
 spectromuzzleuuid UUID
 morecolumns char(1)
}

Table scanspectra {
  sampleuuid UUID [pk]
  subsample char(2) [pk]
  scanuuid UUID
  scanneruuid UUID
  spectromuzzleuuid UUID
  prepcode char(2) [NOT NULL, pk]
  mode varchar(16) [NOT NULL, pk]
  samplerepeats smallint
  darkrepeats smallint
  headtrail smallint
  scattercorrectioncode char[3]
  nafreq real
  negfreq real
  extfreq real
}

Table reflectance {
  scanuuid UUID [pk]
  signalmean real[]
  signalstd real[]
}

Table transmissivity {
  scanuuid UUID [pk]
  signalmean real[]
  signalstd real[]
}

Table fluoresence {
  scanuuid UUID [pk]
  signalmean real[]
  signalstd real[]
}

Table raman {
  scanuuid UUID [pk]
  signalmean real[]
  signalstd real[]
}

Table probes.probe {
  probeuuid UUID [primary key]
  morecolumns char(1)
}

Table scanprobe {
  sampleuuid UUID [pk]
  subsample varchar(8) [pk]
  probeuuid UUID
  proberuuid UUID
  prepcode char(2) [pk]
  mode varchar(16) [pk]
  scanuuid UUID
}

Table proberecord {
  scanuuid UUID [pk]
  registerkey varchar(16) [pk]
  samplerepeats smallint
  darkrepeats smallint
  headtrail smallint
  scattercorrectioncode char[3]
  registervaluemean real
  registervaluestd real
}

Table probeprep {
  prepcode char(2) [NOT NULL, pk]
  sampleprep varchar(32) [NOT NULL]
  info TEXT
  url TEXT
}

Table probemode {
  mode varchar(16) [pk]
  info TEXT
  url TEXT
}

Table support.scattercorrectioncode {
  scattercorrectioncode char[3] [pk]
  scattercorrection TEXT
}

Ref: scanspectra.prepcode - spectraprep.prepcode
Ref: scanspectra.mode - spectramode.mode
Ref: scanprobe.prepcode - probeprep.prepcode
Ref: scanprobe.mode - probemode.mode
Ref: spectrometers.spectromuzzle.spectromuzzleuuid - scanspectra.spectromuzzleuuid
Ref: scanspectra.scanuuid - reflectance.scanuuid
Ref: scanspectra.scanuuid - transmissivity.scanuuid
Ref: scanspectra.scanuuid - fluoresence.scanuuid
Ref: scanspectra.scanuuid - raman.scanuuid
Ref: probes.probe.probeuuid < scanprobe.probeuuid
Ref: scanspectra.scattercorrectioncode -  support.scattercorrectioncode.scattercorrectioncode
Ref: proberecord.scattercorrectioncode -  support.scattercorrectioncode.scattercorrectioncode
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
        "schema": "scans",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "spectramode",
        "command": [
          "mode varchar(16)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "scanspectra",
        "command": [
          "sampleuuid UUID",
          "subsample char(2)",
          "scanuuid UUID",
          "scanneruuid UUID",
          "spectromuzzleuuid UUID",
          "prepcode char(2) [NOT NULL]",
          "mode varchar(16) [NOT NULL]",
          "samplerepeats smallint",
          "darkrepeats smallint",
          "headtrail smallint",
          "scattercorrectioncode char[3]",
          "nafreq real",
          "negfreq real",
          "extfreq real",
          "PRIMARY KEY (sampleuuid, subsample, prepcode, mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "reflectance",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "transmissivity",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "fluoresence",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "raman",
        "command": [
          "scanuuid UUID",
          "signalmean real[]",
          "signalstd real[]",
          "PRIMARY KEY (scanuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "scanprobe",
        "command": [
          "sampleuuid UUID",
          "subsample varchar(8)",
          "probeuuid UUID",
          "prepcode char(2)",
          "proberuuid UUID",
          "mode varchar(16)",
          "scanuuid UUID",
          "PRIMARY KEY (sampleuuid, subsample, prepcode, mode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "proberecord",
        "command": [
          "scanuuid UUID",
          "registerkey varchar(16)",
          "samplerepeats smallint",
          "darkrepeats smallint",
          "headtrail smallint",
          "scattercorrectioncode char[3]",
          "registervaluemean real",
          "registervaluestd real",
          "PRIMARY KEY (scanuuid, registerkey)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "probeprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scans",
        "table": "probemode",
        "command": [
          "mode varchar(16)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (mode)"
        ]
      }
    }
  ]
}
```

### Prepartion codes

TO BE ADDED
