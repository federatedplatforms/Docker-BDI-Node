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
7. Create a zip of the ttl in [this repository](https://github.com/silenroc1/FEDeRATED-copy)
8. Import -> RDF -> Upload the ontology ttl zip
9. Upload file userEvent.shapes.ttl. Target graph, named graph: `http://rdf4j.org/schema/rdf4j#SHACLShapeGraph`
