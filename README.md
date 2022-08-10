# Docker BDI Node

This repository contains the necessary files to configure a BDI node and run it via docker compose.

## Configuration
The BDI node is composed by the following components:
  
  - Corda node
  - Spring-Boot API
  - GraphDB database

### Node Identity

You can configure the identity of the Corda node through the `node.conf` file. You can change the name and location of the organization and you must set the host name or IP address of the node.

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
https://nms.basicdatasharinginfrastructure.net/

## Run the BDI Node

```
docker compose --profile run up
```

Once all the three components are up and running you need to set up the GraphDB repository.

### Set up GraphDB

You can load the ontologies by running the following commands:

1. Navigate to http://\<server>:7200
2. Setup -> Repository -> New free repository
3. Provide a name, put this name in database.properties. The default is bdi.
4. Tick the box to "Enable SHACL validation"
5. Create
6. (Optional) click the thumb-tack icon to set the new repository as the default and run it
7. Download a zip file with the ontology from [here](https://nightly.link/silenroc1/FEDeRATED-Semantic-Model/workflows/CreateArchive/master/federated-ontologies)
8. Import -> RDF -> Upload the ontology zip
9. Download the zip file for the SHACL from [here](https://nightly.link/silenroc1/FEDeRATED-Semantic-Model/workflows/shaclArchive/master/federated-shacl).
10. Import -> RDF -> Upload the shacl zip. Target graph, named graph: `http://rdf4j.org/schema/rdf4j#SHACLShapeGraph`

## Sample Call

Open http://localhost:10050/swagger-ui.html in your browser. A swagger UI should appear.
Under Corda details, one can query the Corda node what nodes it knows. It should know at least one notary (GET `/node/notaries`) and a few other nodes (GET `/node/peers`).

Play around with the `/events` calls too. In case you are prompted for an access token, you can use your iShare instance – make sure you configured it under `database.properties`, together with GraphDB, in case – or enter `Bearer doitanyway` to skip this.

Try to submit the following new events.

```json
{
"fullEvent": "@base <http:\/\/example.com\/base\/> . @prefix pi: <https:\/\/ontology.tno.nl\/logistics\/federated\/PhysicalInfrastructure#> . @prefix classifications: <https:\/\/ontology.tno.nl\/logistics\/federated\/Classifications#> . @prefix dcterms: <http:\/\/purl.org\/dc\/terms\/> . @prefix LogisticsRoles: <https:\/\/ontology.tno.nl\/logistics\/federated\/LogisticsRoles#> . @prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> . @prefix owl: <http:\/\/www.w3.org\/2002\/07\/owl#> . @prefix Event: <https:\/\/ontology.tno.nl\/logistics\/federated\/Event#> . @prefix ReusableTags: <https:\/\/ontology.tno.nl\/logistics\/federated\/ReusableTags#> . @prefix businessService: <https:\/\/ontology.tno.nl\/logistics\/federated\/BusinessService#> . @prefix DigitalTwin: <https:\/\/ontology.tno.nl\/logistics\/federated\/DigitalTwin#> . @prefix skos: <http:\/\/www.w3.org\/2004\/02\/skos\/core#> . @prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> . @prefix ex: <http:\/\/example.com\/base#> . @prefix time: <http:\/\/www.w3.org\/2006\/time#> . @prefix dc: <http:\/\/purl.org\/dc\/elements\/1.1\/> . @prefix era: <http:\/\/era.europa.eu\/ns#> .  ex:Event-b550739e-2ac2-4c21-9a56-e74791313375 a Event:Event, owl:NamedIndividual;   rdfs:label \"GateOut test\", \"Planned gate out\";   Event:hasTimestamp \"2019-09-22T06:00:00Z\"^^xsd:dateTime;   Event:hasDateTimeType Event:Planned;   Event:involvesDigitalTwin ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def, ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:involvesBusinessTransaction ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:hasMilestone Event:START;   Event:hasSubmissionTimestamp \"2019-09-17T23:32:07Z\"^^xsd:dateTime .  ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def a DigitalTwin:TransportMeans,     owl:NamedIndividual .  ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a businessService:Consignment,     owl:NamedIndividual;   businessService:consignmentCreationTime \"2021-05-13T21:23:04Z\"^^xsd:dateTime;   businessService:involvedActor ex:LegalPerson-Maersk .  ex:LegalPerson-Maersk a businessService:LegalPerson, owl:NamedIndividual, businessService:PrivateEnterprise;   businessService:actorName \"Maersk\" .  ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a DigitalTwin:Equipment, owl:NamedIndividual;   rdfs:label \"MNBU0494490\" .",
"countriesInvolved": [

]
}
```

```json
{
  "fullEvent": "@base <http:\/\/example.com\/base\/> . @prefix pi: <https:\/\/ontology.tno.nl\/logistics\/federated\/PhysicalInfrastructure#> . @prefix classifications: <https:\/\/ontology.tno.nl\/logistics\/federated\/Classifications#> . @prefix dcterms: <http:\/\/purl.org\/dc\/terms\/> . @prefix LogisticsRoles: <https:\/\/ontology.tno.nl\/logistics\/federated\/LogisticsRoles#> . @prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> . @prefix owl: <http:\/\/www.w3.org\/2002\/07\/owl#> . @prefix Event: <https:\/\/ontology.tno.nl\/logistics\/federated\/Event#> . @prefix ReusableTags: <https:\/\/ontology.tno.nl\/logistics\/federated\/ReusableTags#> . @prefix businessService: <https:\/\/ontology.tno.nl\/logistics\/federated\/BusinessService#> . @prefix DigitalTwin: <https:\/\/ontology.tno.nl\/logistics\/federated\/DigitalTwin#> . @prefix skos: <http:\/\/www.w3.org\/2004\/02\/skos\/core#> . @prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> . @prefix ex: <http:\/\/example.com\/base#> . @prefix time: <http:\/\/www.w3.org\/2006\/time#> . @prefix dc: <http:\/\/purl.org\/dc\/elements\/1.1\/> . @prefix era: <http:\/\/era.europa.eu\/ns#> .  ex:Event-7f0140f7-1c22-4b68-9bea-25418cd51d18 a Event:Event, owl:NamedIndividual;   rdfs:label \"GateOut test\", \"Planned gate out\";   Event:hasTimestamp \"2019-09-22T06:00:00Z\"^^xsd:dateTime;   Event:hasDateTimeType Event:Planned;   Event:involvesDigitalTwin ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def, ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:involvesBusinessTransaction ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:hasMilestone Event:End;   Event:hasSubmissionTimestamp \"2019-09-17T23:32:07Z\"^^xsd:dateTime .  ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def a DigitalTwin:TransportMeans,     owl:NamedIndividual .  ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a businessService:Consignment,     owl:NamedIndividual;   businessService:consignmentCreationTime \"2021-05-13T21:23:04Z\"^^xsd:dateTime;   businessService:involvedActor ex:LegalPerson-SomeShipper .  ex:LegalPerson-SomeShipper a businessService:LegalPerson, owl:NamedIndividual, businessService:PrivateEnterprise;   businessService:actorName \"SomeShipper\" .  ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a DigitalTwin:Equipment, owl:NamedIndividual;   rdfs:label \"ABCDE\" .",
  "countriesInvolved": [
    
  ]
}
```
