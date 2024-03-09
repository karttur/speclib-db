---
layout: post
title: Schema muzzles
categories: libspec-db
excerpt: "Design and setup for schema muzzles, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-01-09 11:27'
modified: '2024-01-09 T18:17:25.000Z'
comments: true
share: true
---

## Introduktion

A muzzle (snout) is the part of the xspectre [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) spectrometer that contains the lamp and where the sample is placed. The muzzle is attached to the [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) body, in front of the spectral sensor, using a bayonet mount.

This post contains the general design of the schema **muzzles** for the xspectre spectral library and processing system. The design is written in the [Database Markup Language (DBML)](https://dbml.dbdiagram.io/home/). For visualisation of the DBML code I have used the semi free tool [dbdiagram](https://dbdiagram.io/?utm_source=dbml).

## Purpose of the muzzles schema

The purpose of the **muzzles** schema is to allow any combination of lamp (or lamps), spectral methods (diffuse reflectance, transparency, fluorescence and Raman spectroscopy) and type of sample (solid, liquid, gas or plasma), for defining a muzzle model. To accomplish this, the **muzzles** schema contain separate tables for both lamp models and muzzle models, a table itemising all individual muzzles and a range of support tables.

Lamps as such can be of 10 main types:
- laser,
- narrow band LED,
- visible (VIS) broad band LED,
- Near infrared (NIR) broad band LED,
- VIS-NIR broad band LED (combining two LEDs of the two above types),
- MIR broadband LED,
- Tungsten (Halogen-Tungsten) incandescent bulbs,
- Halogen incandescent bulbs,
- Xenon incandescent bulbs, and
- Other (or unknown) incandescent bulb.

As there are many producers, models and versions of each type of lamp listed above, there are many lamps available. But only a few of them are really useful. Lamps that are actually used with [xspectrolum<b>+</b>](https://www.environimagine.com/spectrometerv080.html) must be registered in the table _lampmodel_. The _lampmodel_ table is linked to two support lists:

- public.technology (list of technologies used for generating electromagnetic radiation in a lamp), and
- public.lampinfourl (extended information and url links for each lamp model).

All lamps are mounted in the muzzle via the _xspecled_ Printed Circuit Board (PCB). The small PCB can hold 1 or 2 lamps. The latter is most commonly applied for extending the spectral range of LEDs by combining 2 LEDs, e.g. one LED for VIS and one for NIR. This usually also means that the relative light emission of the two LEDs must be adjusted, which is accomplished by controlling power supply using different resistors. The lamp models, the resistors, the power supply and the PCB version must all be registered in the table _muzzlemodel_ for defining a muzzle.

Additionally, the spectral method and the kind of sample the muzzle is built for also needs to be registered with each muzzle model. This information is then transferred to each individual muzzle when assembled. As the memory capacity attached to each _xspecled_ PCB is limited, the lamp(s), method, state and spectral range of each muzzle model is encoded using an 8-byte string:

- 1: sample state,
- 2: Nr of lamps [1 or 2],
- 3: Spectral method,
- 4: lamp type/band, and
- 5-8: general wavelength band [nm or code]

Code items 1, 3 and 4 are all encoded with a single digit (0-9) and item 2 (nr of lamps) is naturally also a digit. The encoding of 1, 3 and 4 are listed in the following support tables:

- _suooprt.samplestate_ (in the schema **support**),
- _signaltype_ (equivalent to spectral method), and
- _lampband_.

The identification data that is stored in the EEPROM memory of each muzzle is assembled in the table _muzzlecode_. In addition to the identification data, also the power requirements (from the table _muzzlemodel_) is transferred to the EEPROM memory of each muzzle when assembled.

The main table of the **muzzles** schema is the _muzzle_ table. In this table each copy of any muzzle model is registered using a Universally Unique Identifier (uuid) that is registered in the database and written to the small EEPROM memory on the _xspecled_ PCB holding the lamp(s) and that is screwed to the muzzle. The unique _muzzleuuid_ given to each individual muzzle is automatically connected to a sensor (_sensoruuid_) when used for the first time.

### Illustration of the muzzles schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-muzzles.png">
<img src="../../images/DBML_schema-muzzles.png"></a>
<figcaption>Muzzles DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for muzzles'
}

Table lampmodel {
  lampid varchar(36) [primary key]
  source varchar(32)
  product varchar(32)
  model varchar(32)
  mv_min smallint
  mv_max smallint
  mv_typical smallint
  ma_typical smallint
  wl_min smallint
  wl_max smallint
  wl_peak smallint
  technology varchar(16)
  formfactor varchar(16)
  status varchar(1)
}

Table lampinfourl {
 lampid varchar(36)
 info TEXT
 url TEXT
}

Table muzzlemodel {
  muzzlelongid TEXT
  muzzleid varchar(36) [primary key]
  version varchar(8)
  pcb varchar(16)
  lampid1 varchar(36)
  lampid2 varchar(36)
  lampid1resistor integer
  lampid2resistor integer
  mv_min smallint
  mv_max smallint
  mv_typical smallint
  ma_typical smallint
  ms_stabilisationtime smallint
  status varchar(1)
}

Table muzzleinfourl {
 muzzleid varchar(36) [primary key]
 info TEXT
 url TEXT
}

Table muzzlecode {
 muzzleid varchar(36) [primary key]
 muzzleshortid varchar(16)
 samplestatecode char(1)
 nrlamps char(1)
 signaltypecode char(1)
 lampbandcode varchar(4)
 eepromcode varchar(8)
}

Table muzzle {
  muzzleuuid UUID [primary key]
  muzzleid varchar(36)
  serialnr varchar(16)
}

Table support.samplestate {
  samplestatecode char(1) [primary key]
  samplestate varchar(32)
}

Table signaltype {
  signaltypecode char(1) [primary key]
  signaltype varchar(32)
}

Table lampband {
  lampbandcode char(1) [primary key]
  lampband varchar(32)
}

Table technology {
  technology varchar(16) [primary key]
  info TEXT
}

Ref: lampmodel.lampid - lampinfourl.lampid
Ref: lampmodel.technology - technology.technology
Ref: lampmodel.lampid < muzzlemodel.lampid1
Ref: lampmodel.lampid < muzzlemodel.lampid2
Ref: muzzlemodel.muzzleid - muzzleinfourl.muzzleid
Ref: muzzlemodel.muzzleid < muzzle.muzzleid
Ref: muzzlemodel.muzzleid - muzzlecode.muzzleid
Ref: support.samplestate.samplestatecode - muzzlecode.samplestatecode
Ref: signaltype.signaltypecode - muzzlecode.signaltypecode
Ref: lampband.lampbandcode - muzzlecode.lampbandcode
```

## xspeclib code

The xspeclib code can be called by a customised Python script for automatically generating the muzzles schema.

```
{
  "process": [
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "lampmodel",
        "command": [
          "lampid varchar(36)",
          "source varchar(32)",
          "product varchar(32)",
          "model varchar(32)",
          "mv_min smallint",
          "mv_max smallint",
          "mv_typical smallint",
          "ma_typical smallint",
          "wl_min smallint",
          "wl_max smallint",
          "wl_peak smallint",
          "technology varchar(16)",
          "formfactor varchar(16)",
          "status varchar(1)",
          "PRIMARY KEY (lampid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "lampinfourl",
        "command": [
          "lampid varchar(36)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (lampid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "muzzlemodel",
        "command": [
          "muzzlelongid varchar(184)",
          "muzzleid varchar(36)",
          "version varchar(8)",
          "pcb varhar(16)",
          "lampid1 varchar(36)",
          "lampid2 varchar(36) DEFAULT NA",
          "lampid1resistor integer",
          "lampid2resistor integer",
          "mv_min smallint",
          "mv_max smallint",
          "mv_typical smallint",
          "ma_typical smallint",
          "ms_stabilisationtime smallint",
          "status varchar(1)",
          "PRIMARY KEY (muzzleid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "muzzleinfourl",
        "command": [
          "muzzleid varchar(36)",
          "info TEXT",
          "url TEXT",
          "PRIMARY KEY (muzzleid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "muzzlecode",
        "command": [
          "muzzleid varchar(36)",
          "muzzleshortid varchar(16)",
          "samplestatecode char(1)",
          "nrlamps char(1)",
          "signaltypecode char(1)",
          "lampbandcode varchar(4)",
          "eepromcode varchar(8)",
          "PRIMARY KEY (muzzleid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "muzzle",
        "command": [
          "muzzleuuid UUID",
          "muzzleid varchar(36)",
          "serialnr varchar(16)",
          "PRIMARY KEY (muzzleuuid)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "samplestate",
        "command": [
          "samplestatecode char(1)",
          "samplestate varchar(32)",
          "PRIMARY KEY (samplestatecode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "signaltype",
        "command": [
          "signaltypecode char(1)",
          "signaltype varchar(32)",
          "PRIMARY KEY (signaltypecode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
        "table": "lampband",
        "command": [
          "lampbandcode char(1)",
          "lampband varchar(32)",
          "PRIMARY KEY (lampbandcode)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "muzzles",
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
