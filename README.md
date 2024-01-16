# Clearing House
## 1. Objective 
The IDS Clearing House functions as an intermediary within the IDS ecosystem. In this capacity, the IDS Clearing House (CH) 
facilitates communication between a Data Provider (DP) and a Data Consumer (DC), ensuring that both parties fulfill their contractual obligations.
## 2. Leveraging ids-clearing-house-service
To comprehensively understand the functionality of the IDS Clearing House, explore potential enhancements, and gain practical insights, the existing project which is devoped by Fraunhofer AISEC serves as the foundational base.\
The project can be found here: https://github.com/Fraunhofer-AISEC/ids-clearing-house-service/tree/master/clearing-house-app
## 3. Architecture
![image_2024_01_16T08_33_04_057Z](https://github.com/european-dynamics-rnd/Clearing-House/assets/112931086/296ebaf1-7f95-4179-9e3b-d4fac2354b6c)
1. **CH Core Processor:** Is a component that integrates the Clearing House Logging Service into a Connector.
2. **Logging Service:** Is a REST API which depends on two micro services ( **Document API** and **Keyring API**).
3. **Document API:** Is responsible for storing the data and performs basic encryption and decryption for which it depends on the Keyring API.
4. **Keyring API:** Is responsible for creating keys and the actual encryption and decryption of stored data
## 4. Deployment
Prerequisites: 
+ openssl
+ Docker
+ Your favorite IDE for java

Steps:
1. git clone https://github.com/Fraunhofer-AISEC/ids-clearing-house-service.git
2. From https://industrial-data-space.github.io/trusted-connector-documentation/docs/getting_started download the examples zip file.
3. Create a folder named clearing-house 
4. In this folder create an empty docker-compose.yml file.
5. Copy the below code inside docker-compose.yml
```
version: "3.7"
services:
    tc-core-server:
        container_name: "tc-core-server"
        image: fraunhoferaisec/trusted-connector-core:7.2.0
        tty: true
        stdin_open: true
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - ./data/trusted-connector/application.yml:/root/etc/application.yml 
            - ./data/trusted-connector/allow-all-flows.pl:/root/deploy/allow-all-flows.pl
            - ./data/trusted-connector/server-keystore.p12:/root/etc/keystore.p12
            - ./data/trusted-connector/truststore.p12:/root/etc/truststore.p12
        environment:
            TC_DAPS_URL: https://daps-dev.aisec.fraunhofer.de/v4
            SERVICE_SHARED_SECRET: ${SERVICE_SHARED_SECRET}
            TC_CH_ISSUER_CONNECTOR: "https://w3id.org/idsa/core/issuerConnector"
            TC_CH_AGENT: "https://w3id.org/idsa/core/Agent"
            SERVICE_ID_TC: tc-core-server
            SERVICE_ID_LOG: logging-service
        expose:
          - "9999"
          - "8443" 
          - "8080"
          - "29292"
        ports:
            - "8443:8443"
            - "9999:9999"
            - "8080:8080"
            - "29292:29292"
        networks:
            - ch_network
networks:
  ch_network:
    name: ch_network
    driver: bridge
```
6. Create the path which is used for this container as volume  `./data/trusted-connector/ `
7. From deploy folder under trusted-connector example (trusted-connector-examples_latest) copy  `all-all-flows.pl` file to `clearing-house/data/trusted-connector`
8. From `ids-clearing-house-service/docker`  folder copy `application.yml` and move it under `clearing-house/data/trusted-connector`.
9. From `ids-clearing-house-service/clearing-house-processors/src/intTest/resources` copy `truststore.p12` and `sever-keystore.p12`  and move them under `clearing-house/data/trusted-connector`.
10. Add an .env file that will contain the `SERVICE_SHARED_SECRET`
11. Save docker-compose.yml
12. Go to the clearing-house-service repository and open clearing-house-processors project with your IDE and build it. Should be created a jar file named `clearing-house-processors-0.10.0.jar`.
13. Copy this jar under `./data/trusted-connector/`
14. In the clearing-house under trusted-connector folder create a new folder with name `routes`. In clearing-house-processors project there is an xml file under `src/routes` named `clearing-house-routes.xml`. Copy `clearing-house-routes.xml`  to `routes` folder.
15. After that the docker-compose.yml  Should be like:
```
version: "3.7"
services:
    tc-core-server:
        container_name: "tc-core-server"
        image: fraunhoferaisec/trusted-connector-core:7.2.0
        tty: true
        stdin_open: true
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - ./data/trusted-connector/application.yml:/root/etc/application.yml 
            - ./data/trusted-connector/allow-all-flows.pl:/root/deploy/allow-all-flows.pl
            - ./data/trusted-connector/server-keystore.p12:/root/etc/keystore.p12
            - ./data/trusted-connector/truststore.p12:/root/etc/truststore.p12
            - ./data/trusted-connector/clearing-house-processors-0.10.0.jar:/root/jars/clearing-house-processors.jar
            - ./data/trusted-connector/routes/clearing-house-routes.xml:/root/deploy/clearing-house-routes.xml
        environment:
            TC_DAPS_URL: https://daps-dev.aisec.fraunhofer.de/v4
            SERVICE_SHARED_SECRET: ${SERVICE_SHARED_SECRET}
            TC_CH_ISSUER_CONNECTOR: "https://w3id.org/idsa/core/issuerConnector"
            TC_CH_AGENT: "https://w3id.org/idsa/core/Agent"
            SERVICE_ID_TC: tc-core-server
            SERVICE_ID_LOG: logging-service
         expose:
          - "9999"
          - "8443" 
          - "8080"
          - "29292"
        ports:
            - "8443:8443"
            - "9999:9999"
            - "8080:8080"
            - "29292:29292"
        networks:
            - ch_network
networks:
  ch_network:
    name: ch_network
    driver: bridge
```
16. The next step is to build the images for clearing house app part. There are 3 components. `logging-service` `document-api`, `keyring-api` and their databases (mongo)
17. To build images for these components we need to go under `ids-clearing-house-service` folder (the folder that we clone the clearing house repository). There is a directory named `docker`.
18. We should check if the entrypoint.sh is CLF or LF. If is CLF we should change it to LF. This can be done easily using Visual Studio. (This step make sense only if you try to deploy CH in Windows)
19. After that,  we should change some lines of 3  Docker files (document-api-multistage, keyring-api-multistage, logging-service-multistage)
20. For each we should make these changes :
    + In line 6 FROM `ubuntu:20.04` -> TO `ubuntu:22.04`
    + In line 11 && apt-get --no-install-recommends install -y -q ca-certificates gnupg2 libssl1.1 libc6 -> && apt-get --no-install-recommends install -y -q ca-certificates gnupg2 libssl3 libc6
    + Between COPY docker/entrypoint.sh. And ENTRYPOINT ["/server/entrypoint.sh"] add this command: RUN chmod 777 /server/entrypoint.sh
21. Go to the upper folder and  run the below commands to build the  images
     + `docker build -f docker/logging-service-multistage.Dockerfile  . -t logging-service`
     + `docker build -f docker/document-api-multistage.Dockerfile  . -t document-api`
     + `docker build -f docker/keyring-api-multistage.Dockerfile  . -t keyring-api`
22. After that type docker image ls. Should be existed 3 images.
23. In the clearing-house folder we need to create some folders and copy some files from clearing-house-service repository.
24. For now we have under `clearing-house` folder one subfolder named `data`. Inside `data` we have a folder named `trusted-connector`. In the  `data` folder we need to  create the following subfolders: `certs`, `document-api` ,`keyring-api` -> `init_db` (init_db is subfoler of keyring-api folder) ,`keys` ,`mongo`->( `document-api`, `keyring-api`, `logging-service` (3 subfolders under mongo folder)).
25. Let’s copy some mandatory files form clearing-house-service repo to our project.
26. Copy  `ids-clearing-house-service/clearing-house-app/logging-service/Rocket.toml` to `clearing-house/data`
27. Copy  `ids-clearing-house-service/clearing-house-app/document-api/Rocket.toml` to `clearing-house/data/document-api`
28. Copy  `ids-clearing-house-service/clearing-house-app/keyring-api/Rocket.toml` to `clearing-house/data/keyring-api`
29. Copy `ids-clearing-house-service/clearing-house-app/keyring-api/init_db/default_doc_type.json` to  `clearing-house/data/keyring-api/init_db`
30. Copy  `ids-clearing-house-service\clearing-house-app\certs\daps.der` and `ids-clearing-house-service\clearing-house-app\certs\daps-dev.der` to `clearing-house/data/certs`
31. In keys folder we need to create a key. This could be happen if we run the following command : `openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -outform der -out private_key.der`
32. The final docker-compose.yml file should be like the below :
```
version: "3.7"
services:
    tc-core-server:
        container_name: "tc-core-server"
        image: fraunhoferaisec/trusted-connector-core:7.2.0
        tty: true
        stdin_open: true
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - ./data/trusted-connector/application.yml:/root/etc/application.yml 
            - ./data/trusted-connector/allow-all-flows.pl:/root/deploy/allow-all-flows.pl
            - ./data/trusted-connector/server-keystore.p12:/root/etc/keystore.p12
            - ./data/trusted-connector/truststore.p12:/root/etc/truststore.p12
            - ./data/trusted-connector/clearing-house-processors-0.10.0.jar:/root/jars/clearing-house-processors.jar
            - ./data/trusted-connector/routes/clearing-house-routes.xml:/root/deploy/clearing-house-routes.xml
        environment:
            TC_DAPS_URL: https://daps-dev.aisec.fraunhofer.de/v4
            SERVICE_SHARED_SECRET: ${SERVICE_SHARED_SECRET}
            TC_CH_ISSUER_CONNECTOR: "https://w3id.org/idsa/core/issuerConnector"
            TC_CH_AGENT: "https://w3id.org/idsa/core/Agent"
            SERVICE_ID_TC: tc-core-server
            SERVICE_ID_LOG: logging-service

        expose:
          - "9999"
          - "8443" 
          - "8080"
          - "29292"
        ports:
            - "8443:8443"
            - "9999:9999"
            - "8080:8080"
            - "29292:29292"
        networks:
            - ch_network
    
    logging-service:
        container_name: "logging-service"
        image: "logging-service:latest"
        depends_on:
            - document-api
            - keyring-api
            - logging-service-mongo
        environment:
            # Allowed levels: Off, Error, Warn, Info, Debug, Trace
            - API_LOG_LEVEL=Debug
            - SERVICE_ID_LOG=logging-service
            - SERVICE_ID_DOC=document-api
            - SERVICE_ID_KEY=keyring-api
            - SHARED_SECRET=${SERVICE_SHARED_SECRET}
        ports:
            - "8000:8000"
        volumes:
            - ./data/Rocket.toml:/server/Rocket.toml
            - ./data/keys:/server/keys
            - ./data/certs:/server/certs
        networks:
            - ch_network
            
    document-api:
        container_name: "document-api"
        image: "document-api:latest"
        depends_on:
            - keyring-api
            - document-mongo
        environment:
            # Allowed levels: Off, Error, Warn, Info, Debug, Trace
            - API_LOG_LEVEL=Info
            - SERVICE_ID_LOG=logging-service
            - SERVICE_ID_DOC=document-api
            - SERVICE_ID_KEY=keyring-api
            - SHARED_SECRET=${SERVICE_SHARED_SECRET}
        ports:
            - "8001:8001"
        volumes:
            - ./data/document-api/Rocket.toml:/server/Rocket.toml
            - ./data/certs:/server/certs
        networks:
            - ch_network
            
    keyring-api:
        container_name: "keyring-api"
        image: "keyring-api:latest"
        depends_on:
            - keyring-mongo
        environment:
            # Allowed levels: Off, Error, Warn, Info, Debug, Trace
            - API_LOG_LEVEL=Info
            - SERVICE_ID_LOG=logging-service
            - SERVICE_ID_DOC=document-api
            - SERVICE_ID_KEY=keyring-api
            - SHARED_SECRET=${SERVICE_SHARED_SECRET}
        ports:
            - "8002:8002"
        volumes:
            - ./data/keyring-api/init_db:/server/init_db
            - ./data/keyring-api/Rocket.toml:/server/Rocket.toml
            - ./data/certs:/server/certs
        networks:
            - ch_network
         
            
    logging-service-mongo:
        container_name: "logging-service-mongo"
        image: mongo:latest
        environment: 
            MONGO_INITDB_DATABASE: process
        ports:
            - "27019:27017"
        volumes:
            - ./data/mongo/logging-service:/data/db    
        networks:
            - ch_network   
            
    document-mongo:
        container_name: "document-mongo"
        image: mongo:latest
        environment: 
            MONGO_INITDB_DATABASE: document
        ports:
            - "27017:27017"
        volumes:
            - ./data/mongo/document-api:/data/db    
        networks:
            - ch_network 
            
            
    keyring-mongo:
        container_name: "keyring-mongo"
        image: mongo:latest
        environment: 
            MONGO_INITDB_DATABASE: keyring
        ports:
            - "27018:27017"
        volumes:
            - ./data/mongo/keyring-api:/data/db    
        networks:
            - ch_network 
networks:
  ch_network:
    name: ch_network
    driver: bridge
```
The final Tree of CH Project should be like the below:
```
clearing-house
¦   .env
¦   docker-compose.yml
¦   
+---data
    ¦   Rocket.toml
    ¦   
    +---certs
    ¦       daps-dev.der
    ¦       daps.der
    ¦       
    +---document-api
    ¦       Rocket.toml
    ¦       
    +---keyring-api
    ¦   ¦   Rocket.toml
    ¦   ¦   
    ¦   +---init_db
    ¦           default_doc_type.json
    ¦           
    +---keys
    ¦       private_key.der
    ¦       
    +---mongo
    ¦   +---document-api
    ¦   +---keyring-api
    ¦   +---logging-service
    +---trusted-connector
        ¦   allow-all-flows.pl
        ¦   application.yml
        ¦   clearing-house-processors-0.10.0.jar
        ¦   server-keystore.p12
        ¦   truststore.p12
        ¦   
        +---routes
                clearing-house-routes.xml
```
Execute the following command `docker-compose up –d`   in the directory where the docker-compose.yml is located \
For using connector console open a browser and browse to `http://<your_ip>:8080` . The default credentials are username: ids password: ids 
## Clearing House Multipart Interactions
The documentation that describes the communication between an IDS Clearing House and IDS Connector using the Multipart protocol can be found here:\
https://github.com/International-Data-Spaces-Association/IDS-G/tree/main/Communication/protocols/multipart#42-clearing-house-interactions \
An example of Clearing House REST API can be found here: \
https://www.postman.com/crimson-escape-67103/workspace/clearing-house/request/13905546-c4ba3b82-36f1-4131-bd08-4c1ba72a5cb6
