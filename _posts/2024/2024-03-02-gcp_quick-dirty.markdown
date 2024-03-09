---
layout: post
title: GCP spectrum
categories: gcp-db
excerpt: "Design and setup for quick and dirty Google Cloud Platform (GCP) postgreSQL spectral data DB"
tags:
  - db
  - setup
  - gcp
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-02 11:27'
modified: '2024-03-02 T18:17:25.000Z'
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
<a href="../../images/DBML_schema-gcp-quick-dirty.png">
<img src="../../images/DBML_schema-gcp-quick-dirty.png"></a>
<figcaption>Schema for quick and dirty xSpectre GCP database</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  sample: 'PostgreSQL'
  Note: 'Schema for xspectre GCP spectrum DB'
}

Table sample {
  userid UUID
  sampleuuid UUID
  campaignuuid UUID [pk]
  sampledatetime timestamp [pk]
  samplename TEXT
  sampleabbr varchar(16) [pk]
  family varchar(32)
  species varchar(32)
  brand varchar(32)
  version varchar(16)
  longitude double
  latitude double
}

Table spectrometer {
  spectrometeruuid UUID
  sensorid varchar(16) [pk]
  muzzleid varchar(16) [pk]
}

Table sensorwavelengths {
  sensorid varchar(16) [pk]
  wavelengths real[]
}

Table spectra_measurement {
  measurementuuid UUID
  campaignuuid UUID [pk]
  spectrometeruuid UUID [pk]
  sampleuuid UUID [pk]
  depth smallint [pk]
  prepcode char(2) [pk]
  subsample char(2) [pk]
  scandatetime timestamp
  signalmean real[]
  signalstd real[]
  darkmean real[]
  darkstd real[]
}

Table auxil_measurement {
  measurementuuid UUID [pk]
  prepcode char[2] [pk]
  temperature_C real
  moisture_percent real
  salinity_usm real
  nitrogen_mgl real
  phosphorus_mgl real
  potassium_mgl real
  pemetrometer_ph real
  ise_ph real
}

Table whiteref {
  whiterefuuid UUID
  spectrometeruuid UUID [pk]
  scandatetime timestamp [pk]
  signalmean real[]
  signalstd real[]
  darkmean real[]
  darkstd real[]  
}

Table spectraprep {
  prepcode char(2) [pk]
  sampleprep varchar(32) [NOT NULL]
  info TEXT
  url TEXT
}

Table auxilprep {
  prepcode char(2) [pk]
  sampleprep varchar(32) [NOT NULL]
  info TEXT
  url TEXT
}

Ref: "sample"."sampleuuid" < "spectra_measurement"."sampleuuid"

Ref: "spectra_measurement"."prepcode" - "spectraprep"."prepcode"

Ref: "spectrometer"."spectrometeruuid" < "whiteref"."spectrometeruuid"

Ref: "spectrometer"."spectrometeruuid" < "spectra_measurement"."spectrometeruuid"

Ref: "spectrometer"."sensorid" > "sensorwavelengths"."sensorid"

Ref: "spectra_measurement"."measurementuuid" - "auxil_measurement"."measurementuuid"

Ref: "auxil_measurement"."prepcode" - "auxilprep"."prepcode"
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
        "schema": "gcpdraft",
        "table": "sample",
        "command": [
          "userid UUID",
          "sampleuuid UUID",
          "campaignuuid UUID",
          "sampledatetime timestamp",
          "samplename TEXT",
          "sampleabbr varchar(16)",
          "family varchar(32)",
          "species varchar(32)",
          "brand varchar(32)",
          "version varchar(16)",
          "longitude double",
          "latitude double",
          "PRIMARY KEY (campaignuuid,sampledatetime,sampleabbr)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "spectrometer",
        "command": [
          "sensorid varchar(16)",
          "muzzleid varchar(16)",
          "PRIMARY KEY (sensorid, muzzleid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "sensorwavelengths",
        "command": [
          "sensorid varchar(16)",
          "wavelengths real[]",
          "PRIMARY KEY (sensorid"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "spectra_measurement",
        "command": [
          "measurementuuid UUID",
          "campaignuuid UUID",
          "spectrometeruuid UUID",
          "sampleuuid UUID",
          "depth smallint",
          "prepcode char(2)",
          "subsample char(2)",
          "scandatetime timestamp",
          "signalmean real[]",
          "signalstd real[]",
          "darkmean real[]",
          "darkstd real[]",
          "PRIMARY KEY (campaignuuid, spectrometeruuid, sampleuuid, depth,prepcode,subsample)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "auxil_measurement",
        "command": [
          "measurementuuid UUID",
          "prepcode char[2]",
          "temperature_C real",
          "moisture_percent real",
          "salinity_usm real",
          "nitrogen_mgl real",
          "phosphorus_mgl real",
          "potassium_mgl real",
          "pemetrometer_ph real",
          "ise_ph real",
          "PRIMARY KEY (measurementuuid, prepcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "whiteref",
        "command": [
          "whiterefuuid UUID",
          "spectrometeruuid UUID",
          "scandatetime timestamp",
          "signalmean real[]",
          "signalstd real[]",
          "darkmean real[]",
          "darkstd real[]",
          "PRIMARY KEY (spectrometeruuid,scandatetime)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "gcpdraft",
        "table": "spectraprep",
        "command": [
          "prepcode char(2)",
          "sampleprep varchar(32) NOT NULL",
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
        "schema": "gcpdraft",
        "table": "auxilprep",
        "command": [
          "prepcode char(2)",
          "sampleprep varchar(32) NOT NULL",
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
        "table": "scanprobe",
        "command": [
          "sampleuuid UUID",
          "subsample varchar(8)",
          "probeuuid UUID",
          "prepcode char(2)",
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
        "table": "spectramode",
        "command": [
          "mode varchar(2)",
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
