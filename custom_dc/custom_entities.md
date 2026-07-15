---
layout: default
title:  Define custom entities
nav_order: 4
parent: Build your own Data Commons
---

{: .no_toc}
# Define custom (non-place) entities

This page shows you how to define (or extend) custom (non-place) entities, which may be part of the process to add your data to your Custom Data Commons instance. It assumes you are already familiar with the content in [Key concepts](/data_model.html) and [Prepare and load your own data](custom_data.md).

Before creating new entities or entity types, please see [Determine if you need to create new entities](custom_data.md#entities) to determine if you can reuse existing entities and/or entity types from base Data Commons (datacommons.org). 

> **Note**: It is not necessary to create new entities for your Data Commons instance if your data is aggregated by a place type, or your data includes entities that already exist in the base. 

* TOC
{:toc}

## Overview

New _entity types_ are defined in an MCF file. It may be the same file in which you define variables, or it can be a separate one.

New _entities_ (instantiations of a type) can be defined in either MCF or CSV files. If you have thousands of new entities of the same type, you will likely find it much easier to manage their definitions in a CSV file. On this page, we will use CSV for examples, and you can translate them into MCF if you like.

The [directory structure](custom_data.md#dir) is the same as for variables.

In the following sections, we'll describe setting up the non-place entities, as well as how to use them with custom statistical variables. Also see the example files provided in [https://github.com/datacommonsorg/website/tree/master/custom_dc/sample/entities](https://github.com/datacommonsorg/website/tree/master/custom_dc/sample/entities){: target="_blank"}.

## Prepare your data

### Step 1: Define new entity types (if needed)

If you need to define custom [entity types](custom_data.md#entities) in MCF (rare), you define them in MCF. You can have a single MCF file or as many as you like. 

Let's look at a concrete example. We are going to look at hospital data provided by the U.S. Department of Health and Human Services](https://public-data-hub-dhhs.hub.arcgis.com/){: target="_blank"}. The data is aggregated per-hospital, so we'll use the existing [`Hospital`](https://datacommons.org/browser/Hospital){: target="_blank"} class. However, the dataset we'll use, which reports on hospital capacity, reports patient counts. But there is no existing `Patient` class in Data Commons, so we'll create one:

```
Node: dcid:hhs/Patient
typeOf: schema:Class
name: "Patient"
subClassOf: schema:Person
```

For entity types, an MCF block definition must include the following fields:

* `Node`: This is the DCID of the entity or entity type you are defining. DCIDs can be a maximum of 256 characters long. It is also recommended that you use a prefix to create a namespace for your own entity types. The prefix must be separated from the main entity type name by a slash (`/`), and should represent your organization, dataset, project, or whatever makes sense for you. For example, if your organization or project name is "foo.com", you could use a namespace `foo/`. This way it is easy to distinguish your custom entity types from entity types in the base DC.
* `name`: This is the readable name that will be displayed in various parts of the UI. 
* `typeOf`: For an entity type, this must be `Class`.
* `subClassOf`: To link your new entity type to existing types in the knowledge graph, this can be any existing class that is somehow related. This inserts the entity type into a class hierarchy. You may also define sub-types of types you define, by using this field to indicate the "parent" class. In this example, the parent class is `Person`.

You can add other optional properties, such as schema.org meta properties, and any number of key:value pairs.

### Step 1a: Define properties of the entity type (if needed)

In our dataset, hospitals are broken down into 3 subtypes: "long term", "short term" and "critical access". 

 A common way to represent a property like this, where there is a predefined set of possible values and each value is mutually exclusive, is to define an enum. The enum becomes the type of the property, and each member of the enum can be used as a value of the property. Here's an example:

```
# Step 1: Define the enum itself
Node: dcid:HospitalTypeEnum
name: "Hospital subtype enum"
typeOf: schema:Class
subClassOf: schema:Enumeration
description: "Classifies hospitals into different types according to populations served."

# Step 2: Define the members of the enum

Node: dcid:LongTermHospital
name: "Long-term hospital"
typeOf: dcid:HospitalTypeEnum
description: "Hospitals where patient stays are longer than 25 days."

Node: dcid:ShortTermHospital
name: "Short-term hospital"
typeOf: dcid:HospitalTypeEnum
description: "Hospitals where patient stays are shorter than 25 days."

Node: dcid:CriticalAccessHospital
name: "Critical access hospital"
typeOf: dcid:HospitalTypeEnum
description: "Small, rural hospitals with fewer than 25 beds."

# Step 3: Define the property whose values can be of the enum type
Node: dcid:hospitalType
typeOf: dcid:Property
name: "Hospital subtype"
domainIncludes: dcid:Hospital
rangeIncludes: dcid:HospitalTypeEnum
```

These are the important fields to note:
* For the node representing the enum itself, it must be of type `Class` and must be a subclass of `Enumeration`.
* For the nodes representing the allowed values of the enum, they must be of the type you have defined as the enum.
* For the property, it must be of type `Property` and must specify:
  * `domainIncludes`, which specify the entity type to which the property can be applied. In this case, it is any entity of `Hospital` type.
  * `rangeIncludes`, which specifies the allowable types of the property. In this case, it is the hospital type enum.

We'll see some examples of defining non-enum properties later.

{: #step2}
### Step 2: Define new entities

Now let's walk through the process of defining the actual entities you need for your data. You can define entities in both MCF files or CSV files, but we will only provide examples of CSV here. (You can easily convert these to MCF if desired.)

Going back to our example of hospitals in Alaska, although Base Data Commons already has a [`Hospital`](https://datacommons.org/browser/Hospital){: target="_blank"} class, you'll notice that there are no actual hospitals in the knowledge graph. The first step is to add definitions for hospital entities. In the source data, the entities and observations are provided in the same CSV file. But in Data Commons, we need to separate them. Here's how the CSV file might look. The CCN is a certification number that uniquely identifies U.S. hospitals, which will use as the DCIDs. Notice the `hospitalType` column for the property we defined in the previous step.

```csv
ccn,name,address,City,zipCode,hospitalType
hhs/20001,Providence Alaska Medical Center,3200 Providence Drive,geoId/02020,99508,ShortTermHospital
hhs/20008,Bartlett Regional Hospital,3260 Hospital Dr,geoId/02110,99801,ShortTermHospital
hhs/22001,St Elias Specialty Hospital,4800 Cordova Street,geoId/02020,99503,LongTermHospital
hhs/20017,Alaska Regional Hospital,2801 Debarr Road,geoId/02020,99508,ShortTermHospital
hhs/21301,Providence Valdez Medical Center,Po Box 550,geoId/02261,99686,CriticalAccessHospital
hhs/21304,Petersburg Medical Center,Po Box 589,geoId/02280,99833,CriticalAccessHospital
hhs/21306,Providence Kodiak Island Medical Ctr,1915 East Rezanof Drive,geoId/02150,99615,CriticalAccessHospital
hhs/21311,Ketchikan Medical Center,3100 Tongass Avenue,geoId/02150,99901,CriticalAccessHospital
```

A given CSV file can only contain one entity type, so if you are defining entities of more than one type (for example, schools and hospitals), use a separate file for each. 

Here are the important points to note in this example:
* Each entity CSV file can contain as many columns as you need to define various properties of the entity. 
* You must have one column that defines DCIDs for the entities. Here we use the `ccn`.
* Columns can be in any order, with any heading. Even the column defining the DCIDs does not need to be first; you will specify the column to use for DCIDs in `config.json`.
* We recommended that you use a prefix to create a namespace for your own entities. It must be separated from the main variable name by a slash (`/`). For example, if your organization or project name is foo.com, you could use a namespace `foo/`. This way it is easy to distinguish your custom entities from entities in the base DC.
* For any cells that reference existing entities, if you want to link your entities to them, you must specify them by DCID. In the above example, there is a `City` column, that uses the existing [`City`](https://datacommons.org/browser/City){: target="_blank"} DCIDs; in `config.json` we'll declare that column as an existing entity, so that our new hospital entities will be linked to the `City` entity type in the knowledge graph. By contrast, zip codes won't be used to link these entities, so the `zipCode` values aren't given as DCIDs (although they could be).

> **Important:** Whenever you want to link properties of entities you are defining to existing entities, the cell values must contain DCIDs of the relevant entities. If you don't know the DCID, see [Search for an existing entity](custom_data.md#search). 

### Step 3: Write the config.json file

The next step is to create the `config.json` file to configure your new entities. This is the same `config.json` file you use for observations. 

Here's an example of how the file could look for our hospital data.

```json
{
  "inputFiles": {
    "hospital_entities.csv": {
      "importType": "entities",
      "rowEntityType": "Hospital",
      "idColumn": "ccn",
      "entityColumns": [
        "City"
      ],
      "provenance": "Alaska Weekly Hospital Capacity"
    }
  },
  "sources": {
    "HHS Protect Public Data Hub": {
      "url": "https://public-data-hub-dhhs.hub.arcgis.com/",
      "provenances": {
        "Alaska Weekly Hospital Capacity": "https://public-data-hub-dhhs.hub.arcgis.com/datasets/d47bfcaac2544c2eb1fcfb3d36b5ed23_0/explore"
      }
    }
  }
}
```
These are the important fields to note:

* `importType`: By default this is `observations`; to tell the importer that you are adding entities in this CSV file, you must specify `entities`.
* `rowEntityType`: This specifies the entity type that the entities are derived from. In this case, we specify an existing entity type, [`Hospital`](https://datacommons.org/browser/Hospital){: target="_blank"}. Note that the entity type must be identified by its DCID.
* `idColumn`: This indicates to the importer to use the values in the specified column as DCIDs. In this case, we specify `ccn`, which indicates that the values in the `ccn` column should be used as the DCIDs for the entities.
* `entityColumns`: This is optional: if you want properties of your new entities to be linked to existing entities, you can specify the column(s) containing the matching entities. In this case we list the [`City`](https://datacommons.org/browser/City){: target="_blank"} column. Note that the heading of this column must be the DCID of the corresponding entity type, and the values must be the DCIDs of each entity referenced. If you would like the hospitals to be linked by zipcode, you would need to provide the DCID for each zip code.
  
The other fields are explained in the [Data config file specification reference](config.md).

### Step 4: Add statistical variables and observations for new entities

If you are providing observations for the non-place entities, the observations must be in a separate file. You'll need a different CSV file for each entity type for which you are providing observations.

In our dataset, which reports on the 7-day average number of beds, there are the following indicators:

* total_beds_7_day_avg	
* adult_beds_7d_avg	
* adult_inpatient_beds_7d_avg	
* inpatient_beds_occupied_7_day_avg	
* adult_inpatient_beds_occupied_7d_avg

To define these as statistical variables, we first need to find or define entities and properties. It turns out there is already a [Bed](https://datacommons.org/browser/Bed){: target="_blank"} class. But we will likely need to create properties of beds: adult, inpatient, and occupied. Note that each of these can be combined; they are not mutually exclusive. So we need separate properties for each.

```
Node: dcid:adult
typeOf: schema:Property
name: "Adult"
domainIncludes: dcid:Bed

Node: dcid:inpatient
typeOf: schema:Property
name: "Inpatient"
domainIncludes: dcid:Bed

Node: dcid:occupied
typeOf: schema:Property
Name: "Occupied"
domainIncludes: dcid:Bed

Just like for place entities, you provide observations for these variables in a CSV file. The CSV observations file uses the same variable-per-row format and [column headings](custom_data.md#exp-csv) as places. The only difference from a place-based CSV is that the entity column contains the DCIDs of the entities you have defined in a separate CSV (or MCF) file, instead of places. In our example, the DCIDs are the CCNs of the hospitals.

```csv
entity,date,variable,value
20001,2023-01-27,count_staffed_adult_beds,1048
20001,2023-01-27,count_staffed_adult_icu_beds_occupied,146
20001,2023-01-27,count_staffed_adult_inpatient_icu_beds,146
20001,2023-01-27,count_staffed_inpatient_icu_beds,264
20001,2023-01-27,count_staffed_inpatient_icu_beds_occupied,264
20001,2023-01-27,total_count_staffed_beds,1262
20017,2023-01-27,count_staffed_adult_beds,0
20017,2023-01-27,count_staffed_adult_icu_beds_occupied,0
20017,2023-01-27,count_staffed_adult_inpatient_icu_beds,
20017,2023-01-27,count_staffed_inpatient_icu_beds,
20017,2023-01-27,count_staffed_inpatient_icu_beds_occupied,0
21301,2023-01-27,count_staffed_adult_beds,780
21301,2023-01-27,count_staffed_adult_icu_beds_occupied,62
21301,2023-01-27,count_staffed_adult_inpatient_icu_beds,62
21301,2023-01-27,count_staffed_inpatient_icu_beds,101
21301,2023-01-27,count_staffed_inpatient_icu_beds_occupied,66
21301,2023-01-27,total_count_staffed_beds,836
...
```
We could also have added an `observationPeriod` column, which would be set to `P7D` for all rows.

### Step 5: Add the observations CSV to config.json

Now let's update the config file to cover both the entities and the statistical variables. Since there can only be a single `config.json` file, CSV files of observations and entities must be specified in the same config.

```jsonc
{
  "inputFiles": {
    "hospital_entities.csv": {
      "importType": "entities",
      "rowEntityType": "Hospital",
      "idColumn": "ccn",
      "entityColumns": ["City"],
      "provenance": "Alaska Weekly Hospital Capacity"
    },
    "hospital_observations.csv": {
      "importType": "observations",
      "format": "variablePerRow",
      "entityType": "Hospital",
      "provenance": "Alaska Weekly Hospital Capacity"
    }
  },
  "sources": {
    "HHS Protect Public Data Hub": {
      "url": "https://public-data-hub-dhhs.hub.arcgis.com/",
      "provenances": {
        "Alaska Weekly Hospital Capacity": "https://public-data-hub-dhhs.hub.arcgis.com/datasets/d47bfcaac2544c2eb1fcfb3d36b5ed23_0/explore"
      }
    }
  }
}
``` 


## Load your entities data

To load and serve your data locally, see the procedures in [Load local custom data](custom_data.md#loadlocal).

To load data in Google Cloud, see [Load data in Google Cloud](/custom_dc/deploy_cloud.html).

### Verify your entities data

If the servers have started up without errors, check to ensure that your data is showing up as expected.

Non-place entities without observational data are only displayed in the knowledge graph browser. To view your entities in a local server, enter the following in the browser address bar:

<pre>
https://localhost:8080/browser/<var>ENTITY_DCID</var>
</pre>

The _ENTITY_DCID_ is any DCID you have created previously. Using our previous hospitals example, we could enter `https://localhost:8080/browser/AKgov/20017` and would see this:

![](/assets/images/custom_dc/customdc_screenshot12.png){: width="800"}

For an entity type, you will see all the entities you've created as instances of that type listed in the **In Arcs** section, with clickable links. For example:

![](/assets/images/custom_dc/customdc_screenshot13.png){: width="800"}

If you've associated statistical variables with an entity, you will see them at the bottom of the page, with timeline graphs. For example:

![](/assets/images/custom_dc/customdc_screenshot14.png){: width="600"}

See [Verify your data](custom_data.md#verify) for more details on checking variables and observational data.
