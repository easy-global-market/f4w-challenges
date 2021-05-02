# Umbrella repository for the Fiware4Water challenges

## What is in this repository?

In this repository, you'll find:

- One folder per challenge containing scripts in order to inject test data into a context broker
- A [fiware-mlaas-poc](fiware-mlaas-poc) folder containing a BentoML proxy application between a context broker and a ML model deployed in a BentoML container. It also contains some useful scripts to interact with a context broker.
- Two docker-compose files containing the necessary runtime components (a context broker and the BentoML proxy application) to have a FIWARE instance. One flavor (`docker-compose.orionld.yml`) is using Orion-LD context broker, the other one (`docker-compose.stellio.yml`) is using Stellio context broker.

## Scenario for the F4W challenges

Support code for for the BentoML proxy component is in fiware-mlaas-poc.

The scripts to inject the test data for each challenge is in one the following subfolders:

- f4w-challenges-milan
- f4w-challenges-sofia
- f4w-challenges-sww

A JSON-LD context to be used along your requests to the NGSI-LD API exposed by the context brokers is available at: https://raw.githubusercontent.com/easy-global-market/ngsild-api-data-models/feature/mlaas-models/mlaas/jsonld-contexts/mlaas-compound.jsonld

In the [fiware-mlaas-poc](fiware-mlaas-poc) folder, you will also find some Python scripts to help you interacting with a context broker (more details in the README file of the subproject):

- Create a MLModel entity
- Get a list of MLModel entities
- Manage relationships between a “data entity” and one or more MLModel entities (create or delete)
- Create a subscription on new data
- Manually add a new measure on an entity to trigger a notification
- Update properties of a MLModel entity

## Actions to be performed by the challengers

Create a BentoML image containing the trained model, using the BentoML standard and documented process. Deploy the image somewhere at your convenience.

Create a MLModel entity in the CB. The MLModel entity contains (in addition to other attributes defined above) an attribute whose value is the URL of the /predict endpoint natively exposed by BentoML. It can be templated in a simple Python script.

Create a MLProcessing entity referencing the newly created MLModel and providing a subscription query matching the entities where the input data is collected. The BentoML proxy is notified and creates the subscription.

Before publishing new data, update the relationship between the “data entities” and the MLModel entities they want to use for the next predictions:

```json
{
    "id": "urn:ngsi-ld:WaterConsumption:<DMA>",
    "type": "WaterConsumption",
    "dma": {
            "type": "Property",
            "value": "<DMA>"
    },
    "litres": {
            "type": "Property",
            "value": 21345,
            "observedAt": "2021-03-25T09:00:00Z",
            "unitCode": "LTR",
            "period": {
                    "type": "Property",
                    "value": 900,
                    "unitCode": "SEC"
            }
    },
    "activeModels": [
            {
                    "type": "Relationship",
                    "object": "urn:ngsi-ld:MLModel:1234",
                    "datasetId": "urn:ngsi-ld:Dataset:1234"
            },
            {
                    "type": "Relationship",
                    "object": "urn:ngsi-ld:MLModel:5678",
                    "datasetId": "urn:ngsi-ld:Dataset:5678"
            }
    ]
}
```

When data is published into a matching entity, the BentoML component is notified. 

It has to know what data has to be extracted from the payload and where to post it. This info is in the MLModel, so it navigates through all the `activeModels` relationships and for each one, it gets the MLModel entity from the context broker, extracts what it needs and calls the BentoML components.
