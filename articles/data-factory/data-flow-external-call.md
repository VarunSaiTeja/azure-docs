---
title: External call data transformation in mapping data flow
description: Call external custom endpoints for mapping data flows
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.subservice: data-flows
ms.topic: conceptual
ms.date: 11/24/2021
---

# External call transformation in mapping data flow

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[!INCLUDE[data-flow-preamble](includes/data-flow-preamble.md)]

The external call transformation enables data engineers to call out to external REST end points row-by-row in order to add custom or 3rd party results into your data flow streams. 

## Configuration

In the external call transformation configuration panel, you will first pick the type of external endpoint you wish to connect to, then map incoming columns, and finally define an output data structure which will be consumed by downstream transformations.

:::image type="content" source="media/data-flow/external-call-001.png" alt-text="External call":::

### Settings

Choose the inline dataset type and associated linked service. Today, only REST is supported. However, SQL stored procedures and other linked service types will become available as well. See the [REST source configuration](https://docs.microsoft.com/azure/data-factory/connector-rest?tabs=data-factory#source-transformation) for explanations of the settings properties.

### Mapping

You can choose auto-mapping to pass all input columns to the endpoint. Optionally, you can manually set the columns and rename the columns that are sent to the target endpoint here.

### Output

Here is where you will define the data structure for the output of the external call, which will be consumed by downstream data transformations. You can define the data structure manually using ADF data flow syntax to define the column names and data types or click on "import projection" and allow ADF to detect the schema output from the external call. Here is an example schema definition structure as output from a weather REST API GET call:

```
({@context} as string[],
		geometry as (coordinates as string[][][],
		type as string),
		properties as (elevation as (unitCode as string,
		value as string),
		forecastGenerator as string,
		generatedAt as string,
		periods as (detailedForecast as string, endTime as string, icon as string, isDaytime as string, name as string, number as string, shortForecast as string, startTime as string, temperature as string, temperatureTrend as string, temperatureUnit as string, windDirection as string, windSpeed as string)[],
		units as string,
		updateTime as string,
		updated as string,
		validTimes as string),
		type as string)
```

## Examples

### Samples including data flow script

:::image type="content" source="media/data-flow/external-call-002.png" alt-text="External call sample":::

```
source(output(
		id as string
	),
	allowSchemaDrift: true,
	validateSchema: false,
	ignoreNoFilesFound: false) ~> source1
Filter1 call(mapColumn(
		id
	),
	skipDuplicateMapInputs: false,
	skipDuplicateMapOutputs: false,
	output(
		headers as [string,string],
		body as (name as string)
	),
	allowSchemaDrift: true,
	store: 'restservice',
	format: 'rest',
	timeout: 30,
	httpMethod: 'POST',
	entity: 'api/Todo/',
	requestFormat: ['type' -> 'json'],
	responseFormat: ['type' -> 'json', 'documentForm' -> 'documentPerLine']) ~> ExternalCall1
source1 filter(toInteger(id)==1) ~> Filter1
ExternalCall1 sink(allowSchemaDrift: true,
	validateSchema: false,
	skipDuplicateMapInputs: true,
	skipDuplicateMapOutputs: true,
	store: 'cache',
	format: 'inline',
	output: false,
	saveOrder: 1) ~> sink1
```

## Data flow script

```
ExternalCall1 sink(allowSchemaDrift: true,
	validateSchema: false,
	skipDuplicateMapInputs: true,
	skipDuplicateMapOutputs: true,
	store: 'cache',
	format: 'inline',
	output: false,
	saveOrder: 1) ~> sink1
```    

## Next steps

* Use the [Flatten transformation](data-flow-flatten.md) to pivot rows to columns.
* Use the [Derived column transformation](data-flow-derived-column.md) to transform rows.
* See the [REST source](https://docs.microsoft.com/azure/data-factory/connector-rest?tabs=data-factory#source-transformation) for more information on REST settings.
