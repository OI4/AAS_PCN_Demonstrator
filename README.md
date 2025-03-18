# AAS PCN Demonstrator
Demonstrator for the Product Change Notification Use Case. Uses AAS and PCN-Submodel.

### Overview of PCN-Demonstrator
![overview-diagram][overviewDiagram]

### Component Diagram of PCN-Demonstrator
![component-diagram][componentDiagram]

### Requirements
- Linux or WSL
- Docker
- Docker-Compose
- Processor Architecture: amd64 (at the moment not all images are available for arm64 so you cannot run this demonstrator on your Raspberry Pi)

### How to run the PCN-Demonstrator on your machine
1. Copy docker-compose.yaml and the folder "node-red-data" to your machine
2. Run `docker compose up -d` in the folder where docker-compose.yaml is located
3. Wait until all components are up and running. 
4. Add additional user to rabbitMQ by `docker exec -it rabbitmq-broker rabbitmqctl add_user 'rabbit-user' 'JvFNXcxtm5AAh3Wj0yry'`
5. Grant permissions to added rabbitMQ-user by `docker exec -it rabbitmq-broker rabbitmqctl set_permissions -p "/" "rabbit-user" ".*" ".*" ".*"`
5. Install *node-red-dashboard* in the node-red-container by `docker exec -it node-red npm install node-red-dashboard`
6. Restart node-red-container by `docker restart node-red`
7. View your node-RED-Manufacturer-Flow: http://localhost:1880/
8. Import Demo AAS with PNC-Submodel to BaSyx AAS Environment: \
8.1: Open *POST Submodel* Endpoint in Swagger-UI: http://localhost:8081/swagger-ui/index.html#/Submodel%20Repository%20API/postSubmodel \
8.2: Click "Try it out" and copy the json content of [Demo PCN-Submodel](/demo-pcn-submodel.json) to the "Request body"-form and click "Execute". \
8.3: Open *POST AAS* Endpoint in Swagger-UI: http://localhost:8081/swagger-ui/index.html#/Asset%20Administration%20Shell%20Repository%20API/postAssetAdministrationShell \
8.4: Click "Try it out" and copy the json content of [Demo AAS](/demo-AAS.json), which already has a reference to the PCN-submodel, to the "Request body"-form and click "Execute". \
9. Open Demo-AAS in Mnestix Browser and navigate to PCN-Submodel: http://localhost:3000/en/viewer/aHR0cHM6Ly9tZXRhLWxldmVsLmRlL2lkcy9hYXMvNzAzNV8zOTEzXzc1OTFfNjYwMg (a green checkmark in the line of "Connected" visualized the established connection to the MQTT broker.)
10. View your node-RED Dashboard to enter PCN-Record and Description: http://localhost:1880/ui
11. Enter new change record in node-RED Dashboard: http://localhost:1880/ui
12. See change record in Mnestix-Browser - there should be a popup after receiving a message from MQTT broker: http://localhost:3000/en/viewer/aHR0cHM6Ly9tZXRhLWxldmVsLmRlL2lkcy9hYXMvNzAzNV8zOTEzXzc1OTFfNjYwMg

### Provided containers

| Container | Description | URI of running instance |
| ----------- | ----------- | ----------------|
| node-red | App for manufacturer to add new change record to PCN-SM and send MQTT message | http://localhost:1880 / http://localhost:1880/ui|
| rabbitmq-broker | Event broker for exchanging the MQTT messages | http://localhost:1883 
| rabbitmq-broker | RabbitMQ Management Console (username: rabbit-user, PW: JvFNXcxtm5AAh3Wj0yry) | http://localhost:15672
| mnestix-browser | App for customer to subscribe event broker and visualize change record | http://localhost:3000/
| mnestix-api | Backend of Mnestix and proxy to BaSyx AAS services | http://localhost:5064/repo/shells / http://localhost:5064/repo/submodels / http://localhost:5064/swagger/index.html
| aas-environment | BaSyx AAS services to store AASs and submodels | http://localhost:8081/swagger-ui/index.html

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
 "changeRecord":"https://mnestix-basyx-repo-b36325a9.azurewebsites.net/submodels/aHR0cHM6Ly9tZXRhLWxldmVsLmRlL2lkcy9zbS8xODk3XzI1MzdfNTk3Ml8yMjYy/submodel-elements/Records[1]/$value"
 }
 }
```
### Additional infos regarding Mnestix
There is a fork of Mnestix created for supporting the PCN submodel: https://github.com/OI4/Mnestix_PCN_Fork \
The relevant code for evaluating the PCN-Submodel and subscribing the MQTT broker, which was written in the hackathon, is here: https://github.com/OI4/Mnestix_PCN-Fork/tree/pcn_submodel/src/user-plugins/submodels/productChangeNotification \
In the docker-compose file (https://github.com/OI4/AAS_PCN_Demonstrator/blob/main/docker-compose.yaml) the docker image of this Mnestix PCN fork is used. See tag 'oi4-pcn-showcase' on https://hub.docker.com/r/mnestix/mnestix-browser/tags 

[overviewDiagram]: images/pcn-demonstrator-overview.png
[componentDiagram]: images/PCN-Component-Diagram.png
[node-RED-flow]: images/PCN-node-RED-flow.png
[node-RED-ui]: images/PCN-node-RED-ui.png