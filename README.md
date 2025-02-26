# AAS PCN Demonstrator
Demonstrator for the Product Change Notification Use Case. Uses AAS and PCN-Submodel.

### Component Diagram of PCN-Demonstrator
![component-diagram][componentDiagram]

### How to run the PCN-Demonstrator on your machine
1. Copy docker-compose.yaml and the folder "node-red-data" to your machine
2. Run `docker-compose up` in the folder where docker-compose.yaml is located
3. Wait until all components are up and running. 
4. Install *node-red-dashboard* in the node-red-container by `docker exec -it node-red npm install node-red-dashboard`
5. Restart node-red-container by `docker restart node-red`
6. View your node-RED-Manufacturer-Flow: http://localhost:1880/
7. View your node-RED Dashboard to enter PCN-Record and Description: http://localhost:1880/ui
8. **TODO / fix known issues:** 
- use the rabbitMQ docker container inside the docker-compose network (right now the public/online HiveMQ-server is used because of connection problems from node-RED to rabbitMQ in the docker-compose network)
- upload demo AASX on startup of BaSyx environment
- use the Mnestix Browser version which supports the PCN-SM inkluding subscribing to MQTT broker 
- improve the node-RED-flow to write the correct path of the latest change record in the MQTT payload (is hardcoded to index 1 right now)



### Provided containers

| Container | Description | URI of running instance |
| ----------- | ----------- | ----------------|
| node-red | App for manufacturer to add new change record to PCN-SM and send MQTT message | http://localhost:1880 / http://localhost:1880/ui|
| rabbitmq-broker | Event broker for exchanging the MQTT messages | http://localhost:1883
| mnestix-browser | App for customer to subscribe event broker and visualize change record | http://localhost:3000/
| mnestix-api | Backend of Mnestix and proxy to BaSyx AAS services | http://localhost:5064/repo/shells / http://localhost:5064/repo/submodels
| aas-environment | BaSyx AAS services to store AASs and submodels - access via mnestix-api
| aas-discovery | BaSyx component
| aas-registry | BaSyx component
| submodel-registry | BaSyx component

### How to present the PCN-Demonstrator

1. Open the demo AAS with PCN-Submodel in the Mnestix Browser. (Mnestix reads the address of the broker and subscribes to the topic defined in the PCN-SM)
2. [optional] Show the node-RED flow to give an idea what is happening on the manufacturer site. (http://localhost:1880)
![node-RED-flow][node-RED-flow]
3. Open the node-RED-UI, fill the values and click 'UPDATE' (http://localhost:1880/ui)
![node-RED-ui][node-RED-ui]
4. The node-RED-flow is now getting the latest change records from the PCN-SM and adds an additional change record with the values from the node-RED-UI and writes it back to the PCN-SM. Then the MQTT message is sent to the defined topic.
5. Mnestix Browser gets the MQTT message and loads the addressed change record from PCN-SM and visualizes it.

### Decisions made in the demonstator
1. There is just an instance AAS and no type AAS for the asset, so the PCN-submodel is referenced by the instance AAS
2. For demo purposes the topic for exchanging the MQTT message should be human readable and not Base64-encoded. We took the global AssetId and replaced all characters with special meaning in MQTT-topics with an underscore. So the topic is now `oi4-bike_de_ids_asset_9202_4872_3095_1328`.
3. The MQTT payload is not standardized in the PCN-SM-Specification so we decided to transport the SM-Id, the Url of the SM and the path to the relevant change record, e.g.: 
```
{"submodel":
{
 "id":"https://meta-level.de/ids/sm/1897_2537_5972_2262",
 "path":"https://mnestix-basyx-repo-b36325a9.azurewebsites.net/submodels/aHR0cHM6Ly9tZXRhLWxldmVsLmRlL2lkcy9zbS8xODk3XzI1MzdfNTk3Ml8yMjYy",
 "changeRecord":"https://mnestix-basyx-repo-b36325a9.azurewebsites.net/submodels/aHR0cHM6Ly9tZXRhLWxldmVsLmRlL2lkcy9zbS8xODk3XzI1MzdfNTk3Ml8yMjYy/submodel-elements/Records[1]"
 }
 }
```


[componentDiagram]: images/PCN-Component-Diagram.png
[node-RED-flow]: images/PCN-node-RED-flow.png
[node-RED-ui]: images/PCN-node-RED-ui.png