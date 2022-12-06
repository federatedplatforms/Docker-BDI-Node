# Docker BDI Node

This repository contains the necessary files to configure a BDI node and run it via docker compose.

## Configuration
The BDI node is composed by the following components:
  
  - Corda node
  - Spring-Boot API
  - GraphDB database
  - Semantic adapter

### Node Identity

You can configure the identity of the Corda node through the `node.conf` file. You can change the name and location of the organization and you must set the host name or IP address of the node. 
Note that for testing purposes, it is useful to pick a unique country code. The country code does not have to correspond to a real country.

You can also change the user and password, in that case make sure to match the ones mentioned in `docker-compose.yml`, under the `spring` service.

### GraphDB

The Corda node communicates with an instance of GraphDB. In the default configuration it is assumed that GraphDB and the Corda node are running in the same network (in this case the virtual one created by docker). Should that not be the case, you can configure GraphDB's location by editing `database.properties`.

## Registration to the Network

> **Important**: you don't need to run this step if the node is already registered to the network - for instance, if you already registered it and now you are restarting the node.

After configuring your node identity, you can register your node:

```
docker compose --profile registration up
```

Once the process ends successfully, the node is registered to the network and new certificates are created.

After starting your node, you can see the network-map service at:  
https://nms.k8s.basicdatasharinginfrastructure.net/

## Run the Corda migration database

```
docker compose --profile db up
```

## Run the BDI Node

```
docker compose --profile run up
```

Once all the three components are up and running you need to set up the GraphDB repository.

## Sample Call

Open http://localhost:10050/swagger-ui.html in your browser. A swagger UI should appear.
Under Corda details, one can query the Corda node what nodes it knows. It should know at least one notary (GET `/node/notaries`) and a few other nodes (GET `/node/peers`).

Play around with the `/events` calls too. In case you are prompted for an access token, you can use your iShare instance – make sure you configured it under `database.properties`, together with GraphDB, in case – or enter `Bearer doitanyway` to skip this.

Try to submit the following new and randomly event at the `/events/` endpoint.

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

If you want to randomly generate events yourself to input to `/events/` then use the `/events/random` endpoint, mentioning `false` for start-flow and the desired number of events

Alternatively, you can run the curl command below to generate random events (replace the number-events value with the desired number of events to be generated):
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/plain' --header 'Authorization: Bearer doitanyway' 'http://localhost:10050/events/random?start-flow=false&number-events=1'
```
