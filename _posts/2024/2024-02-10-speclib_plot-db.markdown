---
layout: post
title: Scan schema
categories: libspec-db-todo
excerpt: "Design and setup for schema scan, xspectre postgreSQL spectral library"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-02-10 11:27'
modified: '2024-02-10 T18:17:25.000Z'
comments: true
share: true
---

## Introduction



### Illustration of the scan schema

Mouse click on the figure to get a larger illustration in a pop-up window.

<figure>
<a href="../../images/DBML_schema-scan.png">
<img src="../../images/DBML_schema-scan.png"></a>
<figcaption>Scan DBML database structure</figcaption>
</figure>

### DBML Code

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Schema for spectra model library'
}

Table regressorsymbol {
  campaignuuid char(36) [pk]
  regressorcode char(2) [pk]
  matplotlibsymbol varchar(32) [NOT NULL]
  symbolcolor varchar(32)
  symbolsize smallint
}

Table targetfeaturesymbol {
  campaignuuid char(36) [pk]
  targetfeature char(2) [pk]
  matplotlibsymbol varchar(32) [NOT NULL]
  symbolcolor varchar(32)
  symbolsize smallint
}

Table othermatplolibsymbols {

}

Table modelplotresults {
  modeluuid char(36)
  tightLayout boolean
  singles boolean
  multiple boolean

}

Table modelplotresultssingles {
  figsizex real
  figsizey real
  xadd real
  yadd real
  screenshow boolean
  savepng boolean
}

table multiple {
  subfigsizex real
  subfigsizey real
  xadd real
  yadd real
  screenshow boolean
  savepng boolean
}


"plot": {
    "tightLayout": false,
    "singles": {
      "apply": true,
      "figSize": {
        "x": 0,
        "y": 0,
        "xadd": 0.25,
        "yadd": 0.25
      },
      "screenShow": false,
      "savePng": true
    },
    "rows": {
      "apply": true,
      "subFigSize": {
        "x": 3,
        "y": 3,
        "xadd": 0.1,
        "yadd": 0.1
      },
      "screenShow": false,
      "savePng": true,
      "targetFeatures": {
        "apply": true,
        "figSize": {
          "x": 0,
          "y": 0,
          "xadd": 0,
          "yadd": 0
        },
        "hwspace": {
          "hspace": 0.25,
          "wspace": 0.25
        },
        "columns": [
          "permutationImportance",
          "featureImportance",
          "Kfold"
        ]
      },
      "regressionModels": {
        "apply": true,
        "figSize": {
          "x": 0,
          "y": 0,
          "xadd": 0.25,
          "yadd": 0.25
        },
        "hwspace": {
          "hspace": 0.25,
          "wspace": 0.25
        },
        "columns": [
          "permutationImportance",
          "trainTest",
          "Kfold"
        ]
      }
    }
  }
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
        "schema": "scan",
        "table": "subsample",
        "command": [
          "sampleuuid char(36)",
          "subsample char(2)",
          "scandatetime timestamp",
          "PRIMARY KEY (sampleuuid,subsample)"
        ]
      }
    },
    {
      "processid": "createtable",
      "overwrite": false,
      "delete": false,
      "parameters": {
        "db": "speclib",
        "schema": "scan",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info varchar (255)",
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
        "schema": "scan",
        "table": "spectramode",
        "command": [
          "mode varchar(2)",
          "info varchar (255)",
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
        "schema": "scan",
        "table": "scanspectra",
        "command": [
          "sampleuuid char(36)",
          "subsample char(2)",
          "scanuuid char(36)",
          "spectromuzzleuuid char(36)",
          "prepcode char(2) [NOT NULL]",
          "mode varchar(16) [NOT NULL]",
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
        "schema": "scan",
        "table": "reflectance",
        "command": [
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "transmissivity",
        "command": [
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "fluoresence",
        "command": [
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "raman",
        "command": [
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "scanprobe",
        "command": [
          "sampleuuid char(36)",
          "subsample varchar(8)",
          "probeuuid char(36)",
          "prepcode char(2)",
          "mode varchar(16)",
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "proberecord",
        "command": [
          "scanuuid char(36)",
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
        "schema": "scan",
        "table": "spectraprep",
        "command": [
          "prepcode char(2) NOT NULL",
          "sampleprep varchar(32) [NOT NULL]",
          "info varchar (255)",
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
        "schema": "scan",
        "table": "spectramode",
        "command": [
          "mode varchar(2)",
          "info varchar (255)",
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
