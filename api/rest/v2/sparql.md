---
layout: default
title: Query with SPARQL
nav_order: 5
parent: REST (V2)
grand_parent: API - Query data programmatically
published: true
---
{: .no_toc}
# /v2/sparql

This endpoint makes it possible to query the Data Commons knowledge graph using
[SPARQL](https://www.w3.org/TR/rdf-sparql-query/){: target="_blank"}. SPARQL is a query language developed to retrieve data from [RDF graph](https://en.wikipedia.org/wiki/Resource_Description_Framework){: target="_blank"} content on the web. It leverages the graph structure innate in the data it 
queries to return specific information.

> **Note:** Data Commons only supports a limited subset of SPARQL functionality at this time: specifically, only the keywords `WHERE`, `ORDER BY`, `DISTINCT`, and `LIMIT` are supported.

* TOC
{:toc}

## Request

> Note: GET requests are not provided because they are inconvenient to use with SPARQL.

<p class="api-header">
POST request
</p>

<p class="api-syntax-box api-signature">
URL: https://api.datacommons.org/v2/sparql

Header: X-API-Key: AIzaSyCTI4Xz-UW_G2Q2RfknhcfdAnTHq5X5XuI

JSON data: { "query": "<var>SPARQL_QUERY</var>" }
</p>

<script src="/assets/js/syntax_highlighting.js"></script>
<script src="/assets/js/api-doc-tabs.js"></script>

### Query parameters

| Name                                                | Type   | Description                                                                                                                                                     |
| --------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| key <br /> <required-tag>Required</required-tag>    | string | Your API key. See the the section on [authentication](/api/rest/v2/index.html#authentication) for details. |
| query <br /> <required-tag>Required</required-tag> | string | A SPARQL query string.<br/>In the query, all desired entities must be specified; wildcards are not supported. Each node or entity should have a `typeOf` condition, for example, <code>?<var>ENTITY_NAME</var> typeOf City</code>.    |
{: .doc-table }

## Response

The response looks like:

<pre>
{
  "header": [
    <var>STRING</var>
  ],
  "rows": [
    {
      "cells": [
        {
          "value": <var>STRING</var>
        }
      ]
    },
    ...
  ]
}
</pre>
{: .response-signature .scroll}

### Response fields

| Name   | Type   | Description                                                                      |
| ------ | ------ | -------------------------------------------------------------------------------- |
| header | list | List of strings corresponding to the query variables.                            |
| rows   | list | List of `row` objects, with each containing a list of cells and its cell values. |
| cells  | object | Contains string field `value` corresponding to the queried variable.             |
{: .doc-table}

## Examples

### Example 1: Get a list of all cities with a particular property

Retrieve a list of the DCIDs of cities that have the property [`unDataLabel`](https://datacommons.org/browser/unDataLabel){: target="_blank"}.

Request:
{: .example-box-title}

```bash
curl --request POST \
  --url https://api.datacommons.org/v2/sparql \
  --header 'X-API-Key: AIzaSyCTI4Xz-UW_G2Q2RfknhcfdAnTHq5X5XuI' \
  --data '{
            "query": "SELECT DISTINCT ?subject  \
                WHERE { \
                  ?subject unDataLabel ?object . \
                  ?subject typeOf City 
                }
                LIMIT 10"
  }'
```

Response:
{: .example-box-title}

```json
{
   "header" : [
      "?subject"
   ],
   "rows" : [
      {
         "cells" : [
            {
               "value" : "nuts/DED12"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q623736"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q33986"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q221921"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q841241"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q200797"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "undata-geo/C16801100"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "undata-geo/C07501077"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "undata-geo/C23400001"
            }
         ]
      },
      {
         "cells" : [
            {
               "value" : "wikidataId/Q1005090"
            }
         ]
      }
   ]
}
```
{: .example-box-content .scroll}

### Example 2: Get a list of biological specimens

Retrieve a list of 10 biological specimens (DCID: `BiologicalSpecimen`) in
reverse alphabetical order.

Request:
{: .example-box-title}

```bash
curl --request POST \
  --url https://api.datacommons.org/v2/sparql \
  --header 'X-API-Key: AIzaSyCTI4Xz-UW_G2Q2RfknhcfdAnTHq5X5XuI' \
  --data '{
            "query": "SELECT ?name \
                WHERE { \
                  ?biologicalSpecimen typeOf BiologicalSpecimen . \
                  ?biologicalSpecimen name ?name
                }
                ORDER BY DESC(?name)
                LIMIT 10"
}'
```
{: .example-box-content .scroll}

Response:
{: .example-box-title}

```json
{
  "header": ["?name"],
  "rows": [
    {
      "cells": [
        {
          "value": "x Triticosecale"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Silene"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Silene"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Silene"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    },
    {
      "cells": [
        {
          "value": "x Pseudelymus saxicola (Scribn. & J.G.Sm.) Barkworth & D.R.Dewey"
        }
      ]
    }
  ]
}
```
{: .example-box-content .scroll}
