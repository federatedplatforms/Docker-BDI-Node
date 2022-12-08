# Docker BDI Node

This repository contains the necessary files to configure a BDI node and run it via docker compose.

## Components

The BDI node is composed by the following components:
  
  - Corda node
  - BDI API (includes the semantic adapter)
  - GraphDB

```mermaid
graph TD
    API(BDI API) --> SEM(Semantic Adapter)
    API(BDI API) -- TLS/AMQP --> CORDA(Corda Node)
    CORDA --> GRAPHDB(GraphDB)
    CORDA -- TLS/HTTPS --> ISHARE(iSHARE)
```

## Configuration

Overview of the configuration files:

| File                                                   | Description                                                              |
|--------------------------------------------------------|--------------------------------------------------------------------------| 
| [corda/database.properties](corda/database.properties) | Triple store connection properties                                       | 
| [corda/ishare.properties](corda/ishare.properties)     | iSHARE configuration properties                                          |
| [corda/node.conf](corda/node.conf)                     | Corda node.conf configuration file                                       |
| [.env](.env)                                           | Docker compose env properties, contains BDI API configuration properties | 

Make sure to restart the docker containers after changing the properties.

### Node Identity

You can configure the identity of the Corda node through the [node.conf](corda/node.conf) file. You can change the name and location of the organization, and you must set the host name or IP address of the node. 
Note that for testing purposes, it is useful to pick a unique country code. The country code does not have to correspond to a real country.

### GraphDB

The Corda node communicates with an instance of GraphDB. In the default configuration it is assumed that GraphDB and the Corda node are running in the same network (in this case the virtual one created by docker). 
Should this not be the case, you can configure GraphDB's location by editing [corda/database.properties](corda/database.properties) folder, and alter the `triplestore.host` property.

## Registration to the Network

> **Important**: you don't need to run this step if the node is already registered to the network - for instance, if you already registered it, and now you are restarting the node.

After configuring your node identity, you can register your node:

```
docker compose --profile registration up
```

Once the process ends successfully, the node is registered to the network and new certificates are created. After starting your node, you can see the network-map service at: https://nms.k8s.basicdatasharinginfrastructure.net/

## Run the BDI Node

Run the BDI node using the following docker compose command:

```
docker compose --profile run up
```

This will start the BDI-API, Corda node and GraphDB containers. Startup might take some time, so be patient.

## Sample Call

Open http://localhost:10050/swagger-ui.html in your browser. A swagger UI should appear.
Under Corda details, one can query the Corda node what nodes it knows. It should know at least one notary (GET `/node/notaries`) and a few other nodes (GET `/node/peers`).

Play around with the `/events` calls too. 

Try to submit the following new and randomly event at the `/events` endpoint.

```
@base <http://example.com/base/> . 
@base <http://example.com/base/> . 
@prefix owl: <http://www.w3.org/2002/07/owl#> . 
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix data: <http://example.com/base#> .
@prefix ex: <http://example.com/base#> . 
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix time: <http://www.w3.org/2006/time#> . 
@prefix data: <http://example.com/base#> .
@prefix ex: <http://example.com/base#> . 
@prefix Event: <https://ontology.tno.nl/logistics/federated/Event#> . 
@prefix pi: <https://ontology.tno.nl/logistics/federated/PhysicalInfrastructure#> .
@prefix businessService: <https://ontology.tno.nl/logistics/federated/BusinessService#> .
@prefix dt: <https://ontology.tno.nl/logistics/federated/DigitalTwin#> .
@prefix classifications: <https://ontology.tno.nl/logistics/federated/Classifications#> .

        ex:LegalPerson-Mgeuwp a businessService:LegalPerson, owl:NamedIndividual, businessService:PrivateEnterprise;
          businessService:actorName "Mgeuwp" .
            
        ex:Equipment-9d741476-088b-4b44-9665-da44fa1423fa a dt:Equipment, owl:NamedIndividual;
          rdfs:label "TNO-test092022" .
            
        ex:businessTransaction-8c8b907f-b607-4bd4-9731-e2ee99de30da a businessService:Consignment, owl:NamedIndividual;
          businessService:consignmentCreationTime "2022-01-01T00:01:00"^^xsd:dateTime;
          businessService:involvedActor ex:LegalPerson-Mgeuwp .
            
        ex:PhysicalInfrastructure-QDFKJ a pi:Location, owl:NamedIndividual.
         
        ex:dt-19fc393f-c5d2-4d91-859c-418995a14b00 a dt:TransportMeans, owl:NamedIndividual, dt:Vessel;
          rdfs:label "Vessel";
          dt:hasVIN "1118740";
          dt:hasTransportMeansID "1118740" .
             
        ex:Event-65ed9fff-9848-4234-905b-764afd3f5904 a Event:Event, owl:NamedIndividual;
          Event:hasTimestamp "2021-09-23T19:12:55Z"^^xsd:dateTime;
          Event:hasDateTimeType Event:Estimated;
          Event:involvesDigitalTwin ex:dt-19fc393f-c5d2-4d91-859c-418995a14b00, ex:Equipment-9d741476-088b-4b44-9665-da44fa1423fa;
          Event:involvesBusinessTransaction ex:businessTransaction-8c8b907f-b607-4bd4-9731-e2ee99de30da;
          Event:involvesPhysicalInfrastructure ex:PhysicalInfrastructure-QDFKJ;
          Event:hasMilestone Event:Start;
          Event:hasSubmissionTimestamp "2021-08-23T19:12:55Z"^^xsd:dateTime .
```

You can also send these events to other nodes, by mentioning in the endpoint `/events/{destinationOrganisation}/{destinationLocality}/{destinationCountry}` the organization, locality and country of the node you want to send the events to.

If you want to randomly generate events yourself to input to `/events` then use the `/events/random` endpoint, mentioning `false` for start-flow and the desired number of events

Alternatively, you can run the curl command below to generate random events (replace the number-events value with the desired number of events to be generated):
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/plain' 'http://localhost:10050/events/random?start-flow=false&number-events=1'
```


## Run the Corda migration database (optional)

In case you want to rebuild the corda datastore, run the command below. Make sure to stop the corda bdi node first:

```
docker compose --profile db up
```
