# AAS_PCN_Demonstrator
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




[componentDiagram]: images/PCN-Component-Diagram.png