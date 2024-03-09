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



### Illustration of the macrofauna schema

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
  Note: 'Schema for AI4SH macrofauna'
}

Table common.samplelocation {
  sampleuuid UUID [pk]
  longitude double
  latitude double
  elevation real
}

Table common.landscape {
  sampleuuid UUID [pk]
  sampledatetime timestamp [pk]
  landform TEXT
  slope  smallint
  aspect smallint
  landuse TEXT
  agroforesty boolean
  etc TEXT  
}

Table common.users {
  userid UUID [pk]
  otherfields boolean
}

Table monolith {
  datasetuuid UUID [pk]
  sampledatetime timestamp [pk]
  samplename TEXT [pk]
  subsample char(1) [pk]
  userid UUID
  sampleuuid UUID
  accessrights varchar(16) [NOT NULL]
  sample_dim_cm_x smallint [default: 25]
  sample_dim_cm_y smallint [default: 25]
  sample_dim_cm_z smallint [default: 30]
  extraction_method TEXT [default: "hand_sorting"]
  litter_layer boolean [default: True]
  organic_layer boolean [default: False]
  topspil_layer boolean [default: True]
}

Table life_cycle_stage_code {
  life_cycle_stage_code char[1]
  life_cycle_stage varchar(16)
}

// A=adult, C=cocoon, L = Larvae, J=juvenile, N= Nymph, E= Egg, P=Pupa,

Table observation_litter_layer {
  sampleuuid UUID
  taxa TEXT
  life_cycle_stage_code char[1]
  n_specimen smallint
  g_total real  
}

Table observation_organic_layer {
  sampleuuid UUID
  taxa TEXT
  life_cycle_stage_code char[1]
  n_specimen smallint
  g_total real
}

Table observation_topsoil_layer {
  sampleuuid UUID
  taxa TEXT
  life_cycle_stage_code char[1]
  n_specimen smallint
  g_total real
}

Table extraction_method {
  extraction_method varchar(32) [pk]
}

Table accepted_taxa_stage {
  taxa TEXT [pk]
  taxa_alternative TEXT
  life_cycle_stage_code char(1) [default: 'A']
  common_english_name TEXT[]
  taxa_system_level TEXT [pk]
  class TEXT
  order TEXT
  family TEXT
  genus TEXT [default: 'NA']
  species TEXT [default: 'NA']
  info TEXT
  url TEXT
}

Table taxa_system_level {
  taxa_system_level TEXT
}

REF: common.samplelocation.sampleuuid - monolith.sampleuuid
REF: common.landscape.sampleuuid - monolith.sampleuuid
REF: common.users.userid - monolith.userid
REF: monolith.sampleuuid - observation_litter_layer.sampleuuid
REF: monolith.sampleuuid - observation_organic_layer.sampleuuid
REF: monolith.sampleuuid - observation_topsoil_layer.sampleuuid
REF: observation_litter_layer.taxa - accepted_taxa_stage.taxa
REF: observation_organic_layer.taxa - accepted_taxa_stage.taxa
REF: observation_topsoil_layer.taxa - accepted_taxa_stage.taxa
REF: observation_litter_layer.life_cycle_stage_code - accepted_taxa_stage.life_cycle_stage_code
REF: observation_organic_layer.life_cycle_stage_code - accepted_taxa_stage.life_cycle_stage_code
REF: observation_topsoil_layer.life_cycle_stage_code - accepted_taxa_stage.life_cycle_stage_code

Ref: monolith.extraction_method - extraction_method.extraction_method

Ref: life_cycle_stage_code.life_cycle_stage_code - public.observation_topsoil_layer.life_cycle_stage_code

// taxa: acarina
// alternative_taxa: acari
// taxa_system_level: order
// common_english_name: ['mites','ticks']
// class: arachnida
// order: acarina

// taxa araneidae
// alternative_taxa NA
// taxa_system_level: family
// common_english_name ['orb-weaver spiders']
// class arachnida
// order araneidae
// family araneidae

// taxa chilopoda
// alternative_taxa NA
// taxa_system_level: class
// common_english_name ['centipedes']
// class chilopoda
// order NA
// family NA

// taxa Coleoptera
// alternative_taxa NA
// taxa_system_level: order
// common_english_name: ['beetle']
// phylum: Arthropoda
// class insecta
// order Coleoptera
// family NA

// taxa Collembola
// alternative_taxa NA
// taxa_system_level: class
// common_english_name ['springtales']
// phylum: Arthropoda
// class collembola
// order NA
// family NA

// taxa Dictyoptera
// alternative_taxa NA
// taxa_system_level: genus
// common_english_name ['net-winged beetles']
// phylum: Arthropoda
// class: insecta
// order: Coleoptera
// family: Lycidae
// Genus: Dictyoptera

// taxa Diplopoda
// alternative_taxa NA
// taxa_system_level: class
// common_english_name ['millipede']
// phylum: Arthropoda
// class: Diplopoda
// order: NA
// family: NA
// Genus: NA

// taxa Diploura
// alternative_taxa Diplura
// taxa_system_level: order
// common_english_name ['two-pronged bristletails']
// phylum: Arthropoda
// class: Diplura
// order: NA
// family: NA
// Genus: NA

// taxa Diploura
// alternative_taxa Diplura
// taxa_system_level: order
// common_english_name ['two-pronged bristletails']
// phylum: Arthropoda
// class: Diplura
// order: NA
// family: NA
// Genus: NA

// taxa Diptera
// alternative_taxa NA
// taxa_system_level: order
// common_english_name ['fly']
// phylum: Arthropoda
// class: insecta
// order: Diptera
// family: NA
// Genus: NA

// taxa Gasteropoda
// alternative_taxa: Gastropoda
// taxa_system_level: order
// common_english_name ['slugs', 'snails']
// phylum: Arthropoda
// class: insecta
// order: Diptera
// family: NA
// Genus: NA

// taxa Hemiptera
// alternative_taxa: NA
// taxa_system_level: order
// common_english_name ['true bugs']
// phylum: Arthropoda
// class: insecta
// order: Hemiptera
// family: NA
// Genus: NA

// taxa Hymenoptera
// alternative_taxa: NA
// taxa_system_level: order
// common_english_name ['sawfiles','wasps','bees','ants']
// phylum: Arthropoda
// class: insecta
// order: Hymenoptera
// family: NA
// Genus: NA



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
