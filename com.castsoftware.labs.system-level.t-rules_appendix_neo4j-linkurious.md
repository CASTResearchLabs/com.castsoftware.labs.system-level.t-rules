# Neo4J / Linkurious introduction

Neo4J is a graph database server designed to store and query graphs, composed of nodes and edges.

Neo4J querying language is cypher. Neo4J nodes have 0 to many :LABEL, and edges have 0 to 1 :TYPE. Neo4J nodes and types have 0 to many properties.

Linkurious is a visualisation platform feeding on graph database servers, including Neo4J.

## Integration MO
* update
** delivered files
*** 1.export_TRules.bat, 
*** 2.import_TRules.bat, 
*** 3.merge_and_index.TRules.bat, 
*** 3.merge_and_index.TRules.cql
** with 
*** SQL connection, 
*** directory information, 
*** Neo4j database information,
*** AED publishing URL (to support source code access)
** Note: you can check/improve the SQL used for extraction in 1.export_TRules.sql
* run in sequence
** 1.export_TRules.bat, to extract required data from a Dashboard Service and an Analysis Service into CSV files
** 2.import_TRules.bat, to create a new Neo4J database from nodes and edges defined in CSV files  
(warning: running 2.import_TRules.bat requires Neo4j to be stopped and that the Neo4J database doesn't exist yet), 
** 3.merge_and_index.TRules.bat, to fine tune the Neo4J model and create index for improved performances 
(warning: running 3.merge_and_index.TRules.bat requires Neo4j to be stopped)
* start 
** Neo4j
** Linkurious

## Neo4j model

### Nodes:
Source code objects are
* :Object for all regular objects found in the source code, with objectId, objectName, objectFullName, objectType, artifactNature, complexityLevel, fanIn, hitCount, url, ... properties
** :Open for objets considered as open in the unbalanced capability :TRule
** :Close for objets considered as open in the unbalanced capability :TRule
* :TransactionHead for objects found in the source code that are considered as transaction entry-point (AFP context)
* :TransactionData for objects found in the source code that are considered as transaction data entities (AFP context)
* :CriticalViolation are :Object, :TransactionHead, and :TransactionData that are violating a :CriticalRule

Semantic objects are
* :Rule for all quality rules
** :TRule for Labs' T-Rules (i.e., transaction-level rules)
** :TRuled for quality rules that are taken into account in super-additivity :TRule (i.e., severe enough in health factors and technical criteria covered by the :TRule)
** :CriticalRule for quality rules with the critical contribution option set in a technical criterion
** :SecurityRule for quality rules contributing to Security health factor
** :RobustnessRule for quality rules contributing to Robustness health factor
** :EfficiencyRule for quality rules contributing to Efficiency health factor
** :ChangeabilityRule for quality rules contributing to Changeability health factor
** :TransferabilityRule for quality rules contributing to Transferability health factor
* :Transaction for all transactions (AFP context)
* :Module for all AIP module

Violation information for information associated to violations
* :Detail for associated :Object, :TransactionHead, and :TransactionData in case of regular quality rules and unbalanced capability :Trule and for associated :TransactionHead and :TRule in case of super-additivity :TRule 
* :Group for associated :Object, :TransactionHead, and :TransactionData of quality rules reporting groups of objects
* :Path for associated :Object, :TransactionHead, and :TransactionData of quality rules reporting paths of objects

### Edges
Source code edges
* :References with the callType property containing the AIP link type between :Object 
* :Closes to the :Open from the appropriate :Close

Semantic edges are
* Rule
** :Involves to target the :Object, :TransactionHead, and :TransactionData in violation
** :Impacts to identify :Transaction with violations from the :Rule 
** :Summarizes to identify the :Rule whose severe violations within the same :Transaction creates a violation to the :TRule
** :Identifies to identify the :Detail, :Group, or :Path violation information
* Violation information
** :Reports to target associated :Object
** :ImplementedBy to target contained :Object with stepIndex property providing the sequence
** :Groups to target contained :Object
* Transaction
** :ImplementedBy to target implementing :Object
** :AccessesDataFrom to target :TransactionData
** :StartsWith to target the one :TransactionHead
* Module
** :Contains to target contained :Object

## Linkurious (1.4.2) configuration
### Objectives
Detailed configuration will control default
* Nodes and edges colours based on their :LABEL and :TYPE
* (Web Service only) Nodes and edges captions based on their :LABEL and :TYPE
* Nodes icons based on their :LABEL

Not used below:
* Nodes size based on their :LABEL or on a numerical property

### Default settings in Production.json
 
* Production.json
{
 "dataSources": [
  {
   "readOnly": false,
   "graphdb": {
    "vendor": "neo4j",
    "url": "http://127.0.0.1:7474",
    "user": "neo4j",
    "password": "castdb"
   },
   "index": {
    "vendor": "elasticSearch",
    "host": "127.0.0.1",
    "port": 9201,
    "forceReindex": false,
    "dynamicMapping": false,
    "skipEdgeIndexation": false
   }
  }
 ],
 "allSources": {
  "maxPathLength": 20,
  "shortestPathsMaxResults": 10,
  "connectionRetries": 10,
  "pollInterval": 10,
  "indexationChunkSize": 5000,
  "expandThreshold": 50,
  "searchAddAllThreshold": 500,
  "searchThreshold": 3000,
  "minSearchQueryLength": 1,
  "rawQueryTimeout": 60000,
  "defaultFuzziness": 0.9,
  "layoutWorkers": 2
 },
 "db": {
  "name": "linkurious",
  "username": null,
  "password": null,
  "options": {
   "dialect": "sqlite",
   "storage": "server/database.sqlite"
  }
 },
 "server": {
  "listenPort": 3000,
  "listenPortHttps": 3443,
  "clientFolder": "server/public",
  "cookieSecret": "221955d752674f385f4913ca6472a88cbece7fae2e5921c11e87299da9e51a9c",
  "allowOrigin": null,
  "domain": null,
  "useHttps": false,
  "forceHttps": false,
  "certificateFile": null,
  "certificateKeyFile": null,
  "certificatePassphrase": null
 },
 "logger": {
  "level": "error"
 },
 "alert": {
  "maxMatchTTL": 30,
  "maxMatchesLimit": 5000,
  "maxRuntimeLimit": 600000,
  "maxConcurrency": 1
 },
 "auditTrail": {
  "enabled": false,
  "logFolder": "audit-trail",
  "fileSizeLimit": 5242880,
  "strictMode": false,
  "mode": "rw"
 },
 "access": {
  "authRequired": false,
  "dataEdition": true,
  "widget": true,
  "loginTimeout": 3600
 },
 "clientAnalytics": {
  "enabled": false,
  "code": null,
  "domain": "none"
 },
 "leaflet": [
  {
   "name": "Stamen Toner Lite",
   "thumbnail": "/assets/img/Stamen_TonerLite.png",
   "urlTemplate": "http://{s}.tile.stamen.com/toner-lite/{z}/{x}/{y}.png",
   "attribution": "Map tiles by <a href=\"http://stamen.com\">Stamen Design</a>, <a href=\"http://creativecommons.org/licenses/by/3.0\">CC BY 3.0</a> &mdash; Map data &copy; <a href=\"http://www.openstreetmap.org/copyright\">OpenStreetMap</a>",
   "subdomains": null,
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 20
  },
  {
   "overlay": true,
   "name": "Stamen Toner Lines",
   "thumbnail": "",
   "urlTemplate": "http://stamen-tiles-{s}.a.ssl.fastly.net/toner-lines/{z}/{x}/{y}.png",
   "attribution": "Map tiles by <a href=\"http://stamen.com\">Stamen Design</a>, <a href=\"http://creativecommons.org/licenses/by/3.0\">CC BY 3.0</a> &mdash; Map data &copy; <a href=\"http://www.openstreetmap.org/copyright\">OpenStreetMap</a>",
   "subdomains": "abcd",
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 20
  },
  {
   "name": "Esri World Gray Canvas",
   "thumbnail": "/assets/img/Esri_WorldGrayCanvas.png",
   "urlTemplate": "http://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
   "attribution": "Tiles &copy; Esri &mdash; Esri, DeLorme, NAVTEQ",
   "subdomains": null,
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 16
  },
  {
   "name": "OpenStreetMap B&W",
   "thumbnail": "/assets/img/OpenStreetMap_BlackAndWhite.png",
   "urlTemplate": "http://{s}.tiles.wmflabs.org/bw-mapnik/{z}/{x}/{y}.png",
   "attribution": "&copy; <a href=\"http://osm.org/copyright\">OpenStreetMap</a>",
   "subdomains": null,
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 18
  },
  {
   "name": "CartoDB Positron",
   "thumbnail": "/assets/img/CartoDB_Positron.png",
   "urlTemplate": "http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png",
   "attribution": "&copy; <a href=\"http://www.openstreetmap.org/copyright\">OpenStreetMap</a> &copy; <a href=\"http://cartodb.com/attributions\">CartoDB</a>",
   "subdomains": "abcd",
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 19
  },
  {
   "name": "MapBox Light",
   "thumbnail": "/assets/img/MapBox_Light.png",
   "urlTemplate": "https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}",
   "attribution": "Map data &copy; <a href=\"http://openstreetmap.org\">OpenStreetMap</a>, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA</a>, Imagery &copy <a href=\"http://mapbox.com\">Mapbox</a>",
   "subdomains": null,
   "id": "mapbox.light",
   "accessToken": "pk.eyJ1Ijoic2hleW1hbm4iLCJhIjoiY2lqNGZmanhpMDAxaHc4bTNhZGFrcHZleiJ9.VliJNQs7QBK5e5ZmYl9RTw",
   "minZoom": 2,
   "maxZoom": 20
  },
  {
   "name": "MapBox Streets",
   "thumbnail": "/assets/img/MapBox_Streets.png",
   "urlTemplate": "https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}",
   "attribution": "Map data &copy; <a href=\"http://openstreetmap.org\">OpenStreetMap</a>, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA</a>, Imagery &copy <a href=\"http://mapbox.com\">Mapbox</a>",
   "subdomains": null,
   "id": "mapbox.streets",
   "accessToken": "pk.eyJ1Ijoic2hleW1hbm4iLCJhIjoiY2lqNGZmanhpMDAxaHc4bTNhZGFrcHZleiJ9.VliJNQs7QBK5e5ZmYl9RTw",
   "minZoom": 2,
   "maxZoom": 20
  },
  {
   "name": "MapQuestOpen Aerial",
   "thumbnail": "/assets/img/MapQuestOpen_Aerial.jpg",
   "urlTemplate": "http://otile{s}.mqcdn.com/tiles/1.0.0/sat/{z}/{x}/{y}.jpg",
   "attribution": "Tiles Courtesy of <a href=\"http://www.mapquest.com/\">MapQuest</a> &mdash; Portions Courtesy NASA/JPL-Caltech and U.S. Depart. of Agriculture, Farm Service Agency",
   "subdomains": "1234",
   "id": null,
   "accessToken": null,
   "minZoom": 2,
   "maxZoom": 11
  }
 ],
 "sigma": {
  "renderer": {
   "type": "canvas"
  },
  "settings": {
   "defaultEdgeType": "tapered",
   "font": "robotoregular",
   "defaultLabelColor": "#000",
   "defaultLabelSize": 11,
   "labelThreshold": 5,
   "labelAlignment": "center",
   "defaultEdgeLabelSize": 11,
   "edgeLabelThreshold": 4,
   "labelHoverShadow": "",
   "edgeLabelHoverShadow": "",
   "maxNodeLabelLineLength": 35,
   "defaultNodeColor": "#999999",
   "nodeBorderColor": "default",
   "hoverFontStyle": "bold",
   "nodeHoverBorderSize": 2,
   "defaultNodeHoverBorderColor": "#ffffff",
   "nodeActiveColor": "node",
   "defaultNodeActiveColor": "#999999",
   "nodeActiveLevel": 3,
   "nodeActiveBorderSize": 2,
   "nodeActiveOuterBorderSize": 4,
   "defaultNodeActiveBorderColor": "#ffffff",
   "defaultNodeActiveOuterBorderColor": "#f65565",
   "nodeHoverLevel": 1,
   "edgeColor": "default",
   "defaultEdgeColor": "#a9a9a9",
   "edgeHoverExtremities": true,
   "edgeHoverLevel": 1,
   "activeFontStyle": "bold",
   "edgeActiveColor": "default",
   "defaultEdgeActiveColor": "#f65565",
   "edgeActiveLevel": 3,
   "nodeHaloSize": 25,
   "edgeHaloSize": 20,
   "nodeHaloColor": "#ffffff",
   "edgeHaloColor": "#ffffff",
   "nodeHaloStroke": true,
   "nodeHaloStrokeColor": "#a9a9a9",
   "nodeHaloStrokeWidth": 0.5,
   "nodeHaloClustering": false,
   "nodeHaloClusteringMaxRadius": 80,
   "glyphFillColor": "#999999",
   "glyphStrokeColor": "transparent",
   "glyphFont": "robotoregular",
   "glyphThreshold": 2,
   "glyphTextColor": "white",
   "imgCrossOrigin": "anonymous",
   "legendBorderWidth": 0.5,
   "legendBorderRadius": 4,
   "legendBorderColor": "#999999",
   "minNodeSize": 5,
   "maxNodeSize": 5,
   "minEdgeSize": 2,
   "maxEdgeSize": 2,
   "zoomingRatio": 1.382,
   "doubleClickZoomingRatio": 1,
   "zoomMin": 0.1,
   "zoomMax": 5,
   "doubleClickZoomDuration": 0,
   "autoRescale": [
    "nodeSize",
    "edgeSize"
   ],
   "doubleClickEnabled": false,
   "enableEdgeHovering": true,
   "edgeHoverPrecision": 10,
   "approximateLabelWidth": true,
   "nodesPowRatio": 0.8,
   "edgesPowRatio": 0.8,
   "animationsTime": 1000
  }
 },
 "styles": {
  "nodes": {
   "color": {
    "by": "data.categories",
    "scheme": "nodes.qualitative.categories",
    "active": true
   },
   "icon": {
    "by": "data.categories",
    "scheme": "nodes.icons.categories",
    "active": true
   }
  },
  "edges": {
   "color": {
    "by": "data.type",
    "scheme": "edges.qualitative.type",
    "active": true
   }
  }
 },
 "palette": {
  "nodes": {
   "qualitative": {
    "linkurious_def": {
     "12": [
      "#701E30",
      "#00D745",
      "#FA5AE3",
      "#EB9104",
      "#25623B",
      "#3C4382",
      "#E885A2",
      "#9391ED",
      "#2F9539",
      "#C9B528",
      "#8B2E0B",
      "#7BA273"
     ],
     "15": [
      "#CE83CA",
      "#C0DC43",
      "#67D5AA",
      "#CE7C32",
      "#D5B5B5",
      "#5D8D42",
      "#88CBD8",
      "#637F7B",
      "#C6D396",
      "#73DA68",
      "#7E8BC1",
      "#BF6481",
      "#9A7A4E",
      "#CBAF43",
      "#D16255"
     ],
     "20": [
      "#D6C1B0",
      "#9DDD5A",
      "#D06D34",
      "#D28FD8",
      "#5D8556",
      "#71D4C6",
      "#CDCF79",
      "#D8A836",
      "#5E8084",
      "#738ECD",
      "#D36565",
      "#61DC7B",
      "#9B7168",
      "#97C4DE",
      "#A57E42",
      "#D5DA41",
      "#D06B97",
      "#917097",
      "#689534",
      "#90D59B"
     ],
     "40": [
      "#5FDAA2",
      "#DE6FBC",
      "#D4742C",
      "#4EA4D4",
      "#DBE345",
      "#807757",
      "#C9DCD0",
      "#5BA943",
      "#E39084",
      "#816B9F",
      "#DEDA83",
      "#DDB5D0",
      "#897121",
      "#D85573",
      "#8294E9",
      "#508A7A",
      "#92E13E",
      "#D95B4B",
      "#816772",
      "#A75781",
      "#538245",
      "#D7C19B",
      "#DB9B5F",
      "#ABD4A2",
      "#8D9F2C",
      "#67D8C8",
      "#577C94",
      "#E8A22E",
      "#C792DF",
      "#65E175",
      "#A26238",
      "#ABDD7E",
      "#AFC2DD",
      "#A65A59",
      "#DABD3A",
      "#6DCBDB",
      "#AEA45A",
      "#E088AD",
      "#B09898",
      "#7793CD"
     ]
    },
    "categories": {
     "CriticalRule": "#8b0000",
     "TRule": "#8b0000",
     "TRuled": "#8b0000",
     "CriticalViolation": "#8b0000",
     "ViolationPath": "#8b0000",
     "SecurityRule": "#cb2f44",
     "EfficiencyRule": "#f47461",
     "RobustnessRule": "#ffbd84",
     "TransferabilityRule": "#ffe0e0",
     "ChangeabilityRule": "#ffe0e0",
     "SelectedPath": "#e0e0e0",
     "SelectedTransaction": "#e0e0e0",
     "ToProcess": "#d0d0d0",
     "ToEvolve": "#b0b0b0",
     "EvolutionImpacts": "#c0c0c0",
     "Rule": "#ffffe0",
     "Object": "#add8e6",
     "Open": "#880000",
     "Close": "#00ecb5",
     "TransactionData": "#8fa1d0",
     "TransactionHead": "#6f6db9",
     "Transaction": "#483ba2",
     "Module": "#00008b",
     "Detail": "#d3d3d3",
     "Group": "#d3d3d3",
     "Path": "#a9a9a9"
    }
   },
   "sequential": {
    "7": [
     "#161344",
     "#3f1c4c",
     "#632654",
     "#86315b",
     "#a93c63",
     "#cd476a",
     "#f35371"
    ]
   },
   "icons": {
    "categories": {
     "Object": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf121"
     },
     "Rule": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf002"
     },
     "TransactionData": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf1c0"
     },
     "CriticalViolation": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf1e2"
     },
     "CriticalRule": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf0e7"
     },
     "Group": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf0c0"
     },
     "Path": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf018"
     },
     "Detail": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf05a"
     },
     "Open": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf07c"
     },
     "Close": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf07b"
     },
     "Transaction": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf0c0"
     },
     "TransactionHead": {
      "font": "FontAwesome",
      "scale": 1,
      "color": "#fff",
      "content": "\uf007"
     }
    }
   }
  },
  "edges": {
   "qualitative": {
    "linkurious_def": {
     "12": [
      "#701E30",
      "#00D745",
      "#FA5AE3",
      "#EB9104",
      "#25623B",
      "#3C4382",
      "#E885A2",
      "#9391ED",
      "#2F9539",
      "#C9B528",
      "#8B2E0B",
      "#7BA273"
     ],
     "15": [
      "#CE83CA",
      "#C0DC43",
      "#67D5AA",
      "#CE7C32",
      "#D5B5B5",
      "#5D8D42",
      "#88CBD8",
      "#637F7B",
      "#C6D396",
      "#73DA68",
      "#7E8BC1",
      "#BF6481",
      "#9A7A4E",
      "#CBAF43",
      "#D16255"
     ],
     "20": [
      "#D6C1B0",
      "#9DDD5A",
      "#D06D34",
      "#D28FD8",
      "#5D8556",
      "#71D4C6",
      "#CDCF79",
      "#D8A836",
      "#5E8084",
      "#738ECD",
      "#D36565",
      "#61DC7B",
      "#9B7168",
      "#97C4DE",
      "#A57E42",
      "#D5DA41",
      "#D06B97",
      "#917097",
      "#689534",
      "#90D59B"
     ],
     "40": [
      "#5FDAA2",
      "#DE6FBC",
      "#D4742C",
      "#4EA4D4",
      "#DBE345",
      "#807757",
      "#C9DCD0",
      "#5BA943",
      "#E39084",
      "#816B9F",
      "#DEDA83",
      "#DDB5D0",
      "#897121",
      "#D85573",
      "#8294E9",
      "#508A7A",
      "#92E13E",
      "#D95B4B",
      "#816772",
      "#A75781",
      "#538245",
      "#D7C19B",
      "#DB9B5F",
      "#ABD4A2",
      "#8D9F2C",
      "#67D8C8",
      "#577C94",
      "#E8A22E",
      "#C792DF",
      "#65E175",
      "#A26238",
      "#ABDD7E",
      "#AFC2DD",
      "#A65A59",
      "#DABD3A",
      "#6DCBDB",
      "#AEA45A",
      "#E088AD",
      "#B09898",
      "#7793CD"
     ]
    },
    "type": {
     "Impacts": "#e35875",
     "Summarizes": "#f98d9f",
     "Involves": "#f98d9f",
     "Comounds": "#f98d9f",
     "Identifies": "#ffc7c4",
     "References": "#beecb5",
     "SelectedReferences": "#e0e0e0",
     "EvolutionImpacts": "#c0c0c0",
     "StartsWith": "#8ed29e",
     "AccessesDataFrom": "#66b890",
     "UsesDataFrom": "#8ed29e",
     "Groups": "#ffffe0",
     "Contains": "#ffffe0",
     "Closes": "#00ecb5",
     "Reports": "#ffffe0",
     "CoversFrom": "#ffffe0",
     "CoversTo": "#ffffe0",
     "ImplementedBy": "#ffffe0"
    }
   },
   "sequential": {
    "3": [
     "#132b43",
     "#326896",
     "#54aef3"
    ],
    "5": [
     "#132b43",
     "#22486b",
     "#326896",
     "#438ac3",
     "#54aef3"
    ],
    "7": [
     "#132b43",
     "#1d3f5d",
     "#27547a",
     "#326896",
     "#3d7fb5",
     "#4897d4",
     "#54aef3"
    ]
   }
  }
 },
 "version": "1.4.2",
 "firstRun": false
}
 
Default visualisation option settings via Web Services
Get datasource key
 
http://127.0.0.1:3000/api/dataSources GET answer
{
  "sources": [
    {
      "reason": "The data-source is ready.",
      "configIndex": 0,
      "key": "fa9ab401",
      "name": "Database #0",
      "connected": true,
      "state": "ready"
    }
  ]
}
==> fa9ab401 to use in next two WS queries and in PATCH WS query body
Get sandbox ID
http://127.0.0.1:3000/api/fa9ab401/sandbox GET answer
{
  "visualization": {
    "id": 10,
    "folder": -1,
    "updatedAt": "2016-10-05T12:17:19.560Z",
    "layout": {
      "incremental": false
    },
    "nodes": [
    ],
    "filters": [
    ],
    "edges": [
    ],
    "edgeFields": {
      "captions": {
        "References": {
          "properties": [
            "callType"
          ],
          "active": true,
          "displayName": false,
          "name": "References"
        },
        "Contains": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Contains"
        },
        "StartsWith": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "StartsWith"
        },
        "Involves": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Involves"
        },
        "Summarizes": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Summarizes"
        },
        "Closes": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Closes"
        },
        "ImplementedBy": {
          "properties": [
            "stepIndex"
          ],
          "active": true,
          "displayName": true,
          "name": "ImplementedBy"
        },
        "Identifies": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Identifies"
        },
        "Groups": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Groups"
        },
        "Reports": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Reports"
        },
        "Impacts": {
          "properties": [
            "objectCount"
          ],
          "active": true,
          "displayName": true,
          "name": "Impacts"
        },
        "AccessesDataFrom": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "AccessesDataFrom"
        }
      },
      "fields": [
        {
          "name": "objectCount",
          "active": true
        },
        {
          "name": "stepIndex",
          "active": true
        },
        {
          "name": "callType",
          "active": true
        }
      ]
    },
    "title": "SandBox",
    "userId": 1,
    "sandbox": true,
    "mode": "nodelink",
    "geo": {
      "layers": [
      ]
    },
    "sourceKey": "fa9ab401",
    "nodeFields": {
      "captions": {
        "Open": {
          "displayName": true,
          "properties": [
          ],
          "active": true
        },
        "Object": {
          "active": true,
          "properties": [
            "objectName"
          ],
          "displayName": false
        },
        "TransferabilityRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "TRuled": {
          "displayName": true,
          "properties": [
          ],
          "active": true
        },
        "ChangeabilityRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "Close": {
          "displayName": true,
          "properties": [
          ],
          "active": true
        },
        "Path": {
          "active": true,
          "properties": [
            "pathName"
          ],
          "displayName": false
        },
        "TransactionHead": {
          "active": true,
          "properties": [
            "objectName"
          ],
          "displayName": false
        },
        "No category": {
          "active": true,
          "properties": [
          ],
          "displayName": false
        },
        "CriticalViolation": {
          "active": true,
          "properties": [
            "objectName"
          ],
          "displayName": false
        },
        "Transaction": {
          "active": true,
          "properties": [
            "transactionName"
          ],
          "displayName": false
        },
        "SecurityRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "Group": {
          "active": true,
          "properties": [
            "groupName"
          ],
          "displayName": false
        },
        "EfficiencyRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "CriticalRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": true
        },
        "Detail": {
          "active": true,
          "properties": [
            "detailName"
          ],
          "displayName": false
        },
        "Module": {
          "active": true,
          "properties": [
            "moduleName"
          ],
          "displayName": false
        },
        "Rule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "TRule": {
          "displayName": true,
          "properties": [
          ],
          "active": true
        },
        "RobustnessRule": {
          "active": true,
          "properties": [
            "ruleName"
          ],
          "displayName": false
        },
        "TransactionData": {
          "active": true,
          "properties": [
            "objectName"
          ],
          "displayName": false
        }
      },
      "fields": [
        {
          "name": "objectCount",
          "active": true
        },
        {
          "name": "size",
          "active": true
        },
        {
          "name": "transactionName",
          "active": true
        },
        {
          "name": "transactionFullName",
          "active": true
        },
        {
          "name": "transactionId",
          "active": true
        },
        {
          "name": "evolutionStatus",
          "active": true
        },
        {
          "name": "hitCount",
          "active": true
        },
        {
          "name": "ownRbstVI",
          "active": true
        },
        {
          "name": "objectFullName",
          "active": true
        },
        {
          "name": "ownEffVI",
          "active": true
        },
        {
          "name": "cumulatedEffVI",
          "active": true
        },
        {
          "name": "log2hitCount",
          "active": true
        },
        {
          "name": "url",
          "active": true
        },
        {
          "name": "objectType",
          "active": true
        },
        {
          "name": "complexityLevel",
          "active": true
        },
        {
          "name": "cumulatedSecuVI",
          "active": true
        },
        {
          "name": "fanIn",
          "active": true
        },
        {
          "name": "artifactNature",
          "active": true
        },
        {
          "name": "ownSecuVI",
          "active": true
        },
        {
          "name": "objectName",
          "active": true
        },
        {
          "name": "cumulatedRbstVI",
          "active": true
        },
        {
          "name": "objectId",
          "active": true
        },
        {
          "name": "efficiencyCriticalIssuesCount",
          "active": true
        },
        {
          "name": "violationName",
          "active": true
        },
        {
          "name": "efficiencyCriticalIssues",
          "active": true
        },
        {
          "name": "detailName",
          "active": true
        },
        {
          "name": "detailId",
          "active": true
        },
        {
          "name": "severity",
          "active": true
        },
        {
          "name": "VISecuContribution",
          "active": true
        },
        {
          "name": "SecuCritical",
          "active": true
        },
        {
          "name": "RbstCritical",
          "active": true
        },
        {
          "name": "ruleName",
          "active": true
        },
        {
          "name": "superAdditivityTC",
          "active": true
        },
        {
          "name": "ruleId",
          "active": true
        },
        {
          "name": "VIRbstContribution",
          "active": true
        },
        {
          "name": "ruleType",
          "active": true
        },
        {
          "name": "pathName",
          "active": true
        },
        {
          "name": "pathId",
          "active": true
        },
        {
          "name": "groupName",
          "active": true
        },
        {
          "name": "groupId",
          "active": true
        },
        {
          "name": "moduleName",
          "active": true
        },
        {
          "name": "moduleId",
          "active": true
        },
        {
          "name": "EffCritical",
          "active": true
        },
        {
          "name": "VIEffContribution",
          "active": true
        }
      ]
    },
    "alternativeIds": {
    },
    "createdAt": "2016-10-05T07:28:42.739Z",
    "design": {
      "palette": {
        "nodes": {
          "icons": {
            "categories": {
              "Module": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "CriticalViolation": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "CriticalRule": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "TransactionData": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "Object": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "Transaction": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "Group": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "Path": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "Rule": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              },
              "TransactionHead": {
                "scale": 1,
                "content": "?",
                "font": "FontAwesome",
                "color": "#fff"
              }
            }
          },
          "sequential": {
            "7": [
              "#161344",
              "#3f1c4c",
              "#632654",
              "#86315b",
              "#a93c63",
              "#cd476a",
              "#f35371"
            ]
          },
          "qualitative": {
            "linkurious_def": {
              "12": [
                "#701E30",
                "#00D745",
                "#FA5AE3",
                "#EB9104",
                "#25623B",
                "#3C4382",
                "#E885A2",
                "#9391ED",
                "#2F9539",
                "#C9B528",
                "#8B2E0B",
                "#7BA273"
              ],
              "15": [
                "#CE83CA",
                "#C0DC43",
                "#67D5AA",
                "#CE7C32",
                "#D5B5B5",
                "#5D8D42",
                "#88CBD8",
                "#637F7B",
                "#C6D396",
                "#73DA68",
                "#7E8BC1",
                "#BF6481",
                "#9A7A4E",
                "#CBAF43",
                "#D16255"
              ],
              "20": [
                "#D6C1B0",
                "#9DDD5A",
                "#D06D34",
                "#D28FD8",
                "#5D8556",
                "#71D4C6",
                "#CDCF79",
                "#D8A836",
                "#5E8084",
                "#738ECD",
                "#D36565",
                "#61DC7B",
                "#9B7168",
                "#97C4DE",
                "#A57E42",
                "#D5DA41",
                "#D06B97",
                "#917097",
                "#689534",
                "#90D59B"
              ],
              "40": [
                "#5FDAA2",
                "#DE6FBC",
                "#D4742C",
                "#4EA4D4",
                "#DBE345",
                "#807757",
                "#C9DCD0",
                "#5BA943",
                "#E39084",
                "#816B9F",
                "#DEDA83",
                "#DDB5D0",
                "#897121",
                "#D85573",
                "#8294E9",
                "#508A7A",
                "#92E13E",
                "#D95B4B",
                "#816772",
                "#A75781",
                "#538245",
                "#D7C19B",
                "#DB9B5F",
                "#ABD4A2",
                "#8D9F2C",
                "#67D8C8",
                "#577C94",
                "#E8A22E",
                "#C792DF",
                "#65E175",
                "#A26238",
                "#ABDD7E",
                "#AFC2DD",
                "#A65A59",
                "#DABD3A",
                "#6DCBDB",
                "#AEA45A",
                "#E088AD",
                "#B09898",
                "#7793CD"
              ]
            },
            "categories": {
              "TRuled": "#A75781",
              "Object": "#add8e6",
              "SelectedTransaction": "#e0e0e0",
              "EfficiencyPathViolation": "#D95B4B",
              "Path": "#a9a9a9",
              "TransactionHead": "#6f6db9",
              "CriticalViolation": "#8b0000",
              "Transaction": "#483ba2",
              "SecurityRule": "#cb2f44",
              "Group": "#a9a9a9",
              "EfficiencyRule": "#f47461",
              "CriticalRule": "#8b0000",
              "ToEvolve": "#a0a0a0",
              "Detail": "#a9a9a9",
              "Module": "#00008b",
              "Rule": "#ffffe0",
              "SelectedPath": "#c0c0c0",
              "ToProcess": "#a0a0a0",
              "TRule": "#538245",
              "RobustnessRule": "#ffbd84",
              "TransactionData": "#8fa1d0"
            }
          }
        },
        "edges": {
          "sequential": {
            "3": [
              "#132b43",
              "#326896",
              "#54aef3"
            ],
            "5": [
              "#132b43",
              "#22486b",
              "#326896",
              "#438ac3",
              "#54aef3"
            ],
            "7": [
              "#132b43",
              "#1d3f5d",
              "#27547a",
              "#326896",
              "#3d7fb5",
              "#4897d4",
              "#54aef3"
            ]
          },
          "qualitative": {
            "linkurious_def": {
              "12": [
                "#701E30",
                "#00D745",
                "#FA5AE3",
                "#EB9104",
                "#25623B",
                "#3C4382",
                "#E885A2",
                "#9391ED",
                "#2F9539",
                "#C9B528",
                "#8B2E0B",
                "#7BA273"
              ],
              "15": [
                "#CE83CA",
                "#C0DC43",
                "#67D5AA",
                "#CE7C32",
                "#D5B5B5",
                "#5D8D42",
                "#88CBD8",
                "#637F7B",
                "#C6D396",
                "#73DA68",
                "#7E8BC1",
                "#BF6481",
                "#9A7A4E",
                "#CBAF43",
                "#D16255"
              ],
              "20": [
                "#D6C1B0",
                "#9DDD5A",
                "#D06D34",
                "#D28FD8",
                "#5D8556",
                "#71D4C6",
                "#CDCF79",
                "#D8A836",
                "#5E8084",
                "#738ECD",
                "#D36565",
                "#61DC7B",
                "#9B7168",
                "#97C4DE",
                "#A57E42",
                "#D5DA41",
                "#D06B97",
                "#917097",
                "#689534",
                "#90D59B"
              ],
              "40": [
                "#5FDAA2",
                "#DE6FBC",
                "#D4742C",
                "#4EA4D4",
                "#DBE345",
                "#807757",
                "#C9DCD0",
                "#5BA943",
                "#E39084",
                "#816B9F",
                "#DEDA83",
                "#DDB5D0",
                "#897121",
                "#D85573",
                "#8294E9",
                "#508A7A",
                "#92E13E",
                "#D95B4B",
                "#816772",
                "#A75781",
                "#538245",
                "#D7C19B",
                "#DB9B5F",
                "#ABD4A2",
                "#8D9F2C",
                "#67D8C8",
                "#577C94",
                "#E8A22E",
                "#C792DF",
                "#65E175",
                "#A26238",
                "#ABDD7E",
                "#AFC2DD",
                "#A65A59",
                "#DABD3A",
                "#6DCBDB",
                "#AEA45A",
                "#E088AD",
                "#B09898",
                "#7793CD"
              ]
            },
            "type": {
              "References": "#beecb5",
              "Contains": "#ffffe0",
              "Involves": "#f98d9f",
              "StartsWith": "#8ed29e",
              "Summarizes": "#897121",
              "ImplementedBy": "#ffffe0",
              "Identifies": "#ffc7c4",
              "UsesDataFrom": "#8ed29e",
              "Groups": "#ffffe0",
              "Reports": "#ffffe0",
              "Impacts": "#e35875",
              "SelectedReferences": "#beecb5",
              "AccessesDataFrom": "#66b890"
            }
          }
        }
      },
      "styles": {
        "nodes": {
          "color": {
            "scheme": "nodes.qualitative.categories",
            "by": "data.categories",
            "active": true
          },
          "icon": {
            "scheme": "nodes.icons.categories",
            "by": "data.categories",
            "active": true
          }
        },
        "edges": {
          "color": {
            "scheme": "edges.qualitative.type",
            "by": "data.type",
            "active": true
          }
        }
      }
    }
  }
}
==> 10 to be uses in PATCH WS query body
Patch sandbox 
http://127.0.0.1:3000/api/fa9ab401/sandbox PATCH body
{
  "visualization": {
    "id": 10,
    "edgeFields": {
      "captions": {
        "Groups": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Groups"
        },
        "StartsWith": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "StartsWith"
        },
        "Reports": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Reports"
        },
        "Impacts": {
          "properties": [
              "objectCount"
          ],
          "active": true,
          "displayName": true,
          "name": "Impacts"
        },
        "Contains": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Contains"
        },
        "AccessesDataFrom": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "AccessesDataFrom"
        },
        "Identifies": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Identifies"
        },
        "References": {
          "properties": [
              "callType"
          ],
          "active": true,
          "displayName": false,
          "name": "References"
        },
        "Involves": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "Involves"
        },
        "ImplementedBy": {
          "properties": [
              "stepIndex"
          ],
          "active": true,
          "displayName": true,
          "name": "ImplementedBy"
        },
        "UsesDataFrom": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "UsesDataFrom"
        },
        "EvolutionDataImpactsFrom": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "EvolutionDataImpactsFrom"
        },
        "EvolutionDataImpactsTo": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "EvolutionDataImpactsTo"
        },
        "EvolutionTransactionImpactsFrom": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "EvolutionTransactionImpactsFrom"
        },
        "EvolutionTransactionImpactsTo": {
          "properties": [
          ],
          "active": true,
          "displayName": true,
          "name": "EvolutionTransactionImpactsTo"
        }
      },
      "fields": [
        {
          "name": "objectCount",
          "active": true
        },
        {
          "name": "stepIndex",
          "active": true
        },
        {
          "name": "callType",
          "active": true
        }       
      ]
    },
    "userId": 1,
    "sandbox": true,
    "sourceKey": "fa9ab401",
    "nodeFields": {
      "captions": {
        "Transaction": {
          "displayName": false,
          "properties": [
            "transactionName"
          ],
          "active": true
        },
        "ChangeabilityRule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "ToProcess": {
          "displayName": true,
          "properties": [
              "transactionName",
              "objectName"
          ],
          "active": true
        },
        "RobustnessRule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "SecurityRule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "Object": {
          "displayName": false,
          "properties": [
              "objectName"
          ],
          "active": true
        },
        "Module": {
          "displayName": false,
          "properties": [
              "moduleName"
          ],
          "active": true
        },
        "EfficiencyRule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "TransactionHead": {
          "displayName": false,
          "properties": [
            "objectName"
          ],
          "active": true
        },
        "No category": {
          "displayName": false,
          "properties": [
          ],
          "active": true
        },
        "CriticalViolation": {
          "displayName": false,
          "properties": [
            "objectName"
          ],
          "active": true
        },
        "EfficiencyPathViolation": {
          "displayName": false,
          "properties": [
            "violationName"
          ],
          "active": true
        },
        "TransferabilityRule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "Path": {
          "displayName": false,
          "properties": [
              "pathName"
          ],
          "active": true
        },
        "CriticalRule": {
          "displayName": true,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "Detail": {
          "displayName": false,
          "properties": [
              "detailName"
          ],
          "active": true
        },
        "Group": {
          "displayName": false,
          "properties": [
              "groupName"
          ],
          "active": true
        },
        "TransactionData": {
          "displayName": false,
          "properties": [
            "objectName"
          ],
          "active": true
        },
        "Rule": {
          "displayName": false,
          "properties": [
              "ruleName"
          ],
          "active": true
        },
        "EvolutionDataImpacts": {
            "displayName": false,
             "properties": [
                  "name"
            ],
            "active": true
        },
        "EvolutionTransactionImpacts": {
            "displayName": false,
             "properties": [
                  "name"
            ],
            "active": true
        }
      },
      "fields": [
        {
          "name": "objectCount",
          "active": true
        },
        {
          "name": "size",
          "active": true
        },
        {
          "name": "transactionName",
          "active": true
        },
        {
          "name": "transactionFullName",
          "active": true
        },
        {
          "name": "transactionId",
          "active": true
        },
        {
          "name": "evolutionStatus",
          "active": true
        },
        {
          "name": "hitCount",
          "active": true
        },
        {
          "name": "ownRbstVI",
          "active": true
        },
        {
          "name": "objectFullName",
          "active": true
        },
        {
          "name": "ownEffVI",
          "active": true
        },
        {
          "name": "cumulatedEffVI",
          "active": true
        },
        {
          "name": "log2hitCount",
          "active": true
        },
        {
          "name": "url",
          "active": true
        },
        {
          "name": "objectType",
          "active": true
        },
        {
          "name": "complexityLevel",
          "active": true
        },
        {
          "name": "cumulatedSecuVI",
          "active": true
        },
        {
          "name": "fanIn",
          "active": true
        },
        {
          "name": "artifactNature",
          "active": true
        },
        {
          "name": "ownSecuVI",
          "active": true
        },
        {
          "name": "objectName",
          "active": true
        },
        {
          "name": "cumulatedRbstVI",
          "active": true
        },
        {
          "name": "objectId",
          "active": true
        },
        {
          "name": "efficiencyCriticalIssuesCount",
          "active": true
        },
        {
          "name": "violationName",
          "active": true
        },
        {
          "name": "efficiencyCriticalIssues",
          "active": true
        }
      ]
    },
    "design": {
      "palette": {
        "nodes": {
          "icons": {
            "categories": {
              "CriticalViolation": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "TransactionData": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "CriticalRule": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Object": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Module": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Transaction": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Group": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Path": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "Rule": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              },
              "TransactionHead": {
                "color": "#fff",
                "content": "?",
                "font": "FontAwesome",
                "scale": 1
              }
            }
          },
          "sequential": {
            "7": [
              "#161344",
              "#3f1c4c",
              "#632654",
              "#86315b",
              "#a93c63",
              "#cd476a",
              "#f35371"
            ]
          },
          "qualitative": {
            "linkurious_def": {
              "12": [
                "#701E30",
                "#00D745",
                "#FA5AE3",
                "#EB9104",
                "#25623B",
                "#3C4382",
                "#E885A2",
                "#9391ED",
                "#2F9539",
                "#C9B528",
                "#8B2E0B",
                "#7BA273"
              ],
              "15": [
                "#CE83CA",
                "#C0DC43",
                "#67D5AA",
                "#CE7C32",
                "#D5B5B5",
                "#5D8D42",
                "#88CBD8",
                "#637F7B",
                "#C6D396",
                "#73DA68",
                "#7E8BC1",
                "#BF6481",
                "#9A7A4E",
                "#CBAF43",
                "#D16255"
              ],
              "20": [
                "#D6C1B0",
                "#9DDD5A",
                "#D06D34",
                "#D28FD8",
                "#5D8556",
                "#71D4C6",
                "#CDCF79",
                "#D8A836",
                "#5E8084",
                "#738ECD",
                "#D36565",
                "#61DC7B",
                "#9B7168",
                "#97C4DE",
                "#A57E42",
                "#D5DA41",
                "#D06B97",
                "#917097",
                "#689534",
                "#90D59B"
              ],
              "40": [
                "#5FDAA2",
                "#DE6FBC",
                "#D4742C",
                "#4EA4D4",
                "#DBE345",
                "#807757",
                "#C9DCD0",
                "#5BA943",
                "#E39084",
                "#816B9F",
                "#DEDA83",
                "#DDB5D0",
                "#897121",
                "#D85573",
                "#8294E9",
                "#508A7A",
                "#92E13E",
                "#D95B4B",
                "#816772",
                "#A75781",
                "#538245",
                "#D7C19B",
                "#DB9B5F",
                "#ABD4A2",
                "#8D9F2C",
                "#67D8C8",
                "#577C94",
                "#E8A22E",
                "#C792DF",
                "#65E175",
                "#A26238",
                "#ABDD7E",
                "#AFC2DD",
                "#A65A59",
                "#DABD3A",
                "#6DCBDB",
                "#AEA45A",
                "#E088AD",
                "#B09898",
                "#7793CD"
              ]
            },
            "categories": {
              "Transaction": "#483ba2",
              "SelectedTransaction": "#e0e0e0",
              "TransactionData": "#8fa1d0",
              "TransactionHead": "#6f6db9",
              "ToProcess": "#a0a0a0",
              "SecurityRule": "#cb2f44",
              "RobustnessRule": "#ffbd84",
              "EfficiencyRule": "#f47461",
              "Rule": "#ffffe0",
              "Object": "#add8e6",
              "ToEvolve": "#a0a0a0",
              "SelectedPath": "#c0c0c0",
              "Module": "#00008b",
              "CriticalRule": "#8b0000",
              "CriticalViolation": "#8b0000",
              "EfficiencyPathViolation": "#D95B4B",
              "Detail": "#a9a9a9",
              "Path": "#a9a9a9",
              "Group": "#a9a9a9"
            }
          }
        },
        "edges": {
          "sequential": {
            "3": [
              "#132b43",
              "#326896",
              "#54aef3"
            ],
            "5": [
              "#132b43",
              "#22486b",
              "#326896",
              "#438ac3",
              "#54aef3"
            ],
            "7": [
              "#132b43",
              "#1d3f5d",
              "#27547a",
              "#326896",
              "#3d7fb5",
              "#4897d4",
              "#54aef3"
            ]
          },
          "qualitative": {
            "linkurious_def": {
              "12": [
                "#701E30",
                "#00D745",
                "#FA5AE3",
                "#EB9104",
                "#25623B",
                "#3C4382",
                "#E885A2",
                "#9391ED",
                "#2F9539",
                "#C9B528",
                "#8B2E0B",
                "#7BA273"
              ],
              "15": [
                "#CE83CA",
                "#C0DC43",
                "#67D5AA",
                "#CE7C32",
                "#D5B5B5",
                "#5D8D42",
                "#88CBD8",
                "#637F7B",
                "#C6D396",
                "#73DA68",
                "#7E8BC1",
                "#BF6481",
                "#9A7A4E",
                "#CBAF43",
                "#D16255"
              ],
              "20": [
                "#D6C1B0",
                "#9DDD5A",
                "#D06D34",
                "#D28FD8",
                "#5D8556",
                "#71D4C6",
                "#CDCF79",
                "#D8A836",
                "#5E8084",
                "#738ECD",
                "#D36565",
                "#61DC7B",
                "#9B7168",
                "#97C4DE",
                "#A57E42",
                "#D5DA41",
                "#D06B97",
                "#917097",
                "#689534",
                "#90D59B"
              ],
              "40": [
                "#5FDAA2",
                "#DE6FBC",
                "#D4742C",
                "#4EA4D4",
                "#DBE345",
                "#807757",
                "#C9DCD0",
                "#5BA943",
                "#E39084",
                "#816B9F",
                "#DEDA83",
                "#DDB5D0",
                "#897121",
                "#D85573",
                "#8294E9",
                "#508A7A",
                "#92E13E",
                "#D95B4B",
                "#816772",
                "#A75781",
                "#538245",
                "#D7C19B",
                "#DB9B5F",
                "#ABD4A2",
                "#8D9F2C",
                "#67D8C8",
                "#577C94",
                "#E8A22E",
                "#C792DF",
                "#65E175",
                "#A26238",
                "#ABDD7E",
                "#AFC2DD",
                "#A65A59",
                "#DABD3A",
                "#6DCBDB",
                "#AEA45A",
                "#E088AD",
                "#B09898",
                "#7793CD"
              ]
            },
            "type": {
              "References": "#beecb5",
              "SelectedReferences": "#beecb5",
              "Groups": "#ffffe0",
              "Reports": "#ffffe0",
              "ImplementedBy": "#ffffe0",
              "Contains": "#ffffe0",
              "AccessesDataFrom": "#66b890",
              "UsesDataFrom": "#8ed29e",
              "StartsWith": "#8ed29e",
              "Impacts": "#e35875",
              "Identifies": "#ffc7c4",
              "Involves": "#f98d9f"
            }
          }
        }
      },
      "styles": {
        "nodes": {
          "color": {
            "scheme": "nodes.qualitative.categories",
            "active": true,
            "by": "data.categories"
          },
          "icon": {
            "scheme": "nodes.icons.categories",
            "active": true,
            "by": "data.categories"
          }
        },
        "edges": {
          "color": {
            "scheme": "edges.qualitative.type",
            "active": true,
            "by": "data.type"
          }
        }
      }
    }
  }
}
