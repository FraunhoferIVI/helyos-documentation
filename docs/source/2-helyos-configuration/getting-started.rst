Getting Started 
+++++++++++++++

Utilizing the Docker image provides the simplest method for running helyOS. T
his image can operate locally or be deployed via a cloud provider. 
For convenience, all components and environment variables should be declared within the docker-compose.yml file

Environment Variables
---------------------

Database connection
===================

    helyOS saves all the data in a PostgreSQL database. The database can be hosted in the same server as helyOS or in a different server.

- PGHOST: hostname for postgres.
- PGPORT: connecting port of the postgres.
- PGDATABASE: database name of your application. A postgres instance can host multiple databases.
- PGUSER: username for postgres database.
- PGPASSWORD: password for postgres.

GraphQL Interface
=================
    The GraphQL server is the main entry point for applications or use interfaces. 
    GraphQL requests allow to query and mutate the data in the database.
    It also controls the authentitcation and authorization of apps in helyOS.
    

- GQLPORT: server port to query the GraphQL server (defaults to 5000).
- JWT_SECRET: secret key to encrypt the JWT token.


RabbitMQ connection
===================
    RabbitMQ is the message broker used by helyOS to communicate with the agents. 
    helyOS core does not only publish and consume messages from the RabbitMQ server, but also creates the necessary accounts, topic exchanges and queues.
    
- RABBITMQHOST: hostname for rabbitmq server.
- RABBITMQPORT: connecting port for AMQP clients.
- RBMQ_API_PORT:  configuration port for REST API for rabbitmq server. It is used to create the rabbitmq accounts.     
- RBMQ_SSL= True or False.  If True, the AMQP connection to rabbitmq server is encrypted using TSL.
- RBMQ_API_SSL= True or False.  If True, the API connection to rabbitmq server encrypted (default = RBMQ_SSL ).

- CREATE_RBMQ_ACCOUNTS: True or False.  helyOS automatically creates the rabbitmq accounts 
- RBMQ_ADMIN_USERNAME: rabbitmq admin username (required if CREATE_RBMQ_ACCOUNTS is True)
- RBMQ_ADMIN_PASSWORD: rabbitmq admin password (required if CREATE_RBMQ_ACCOUNTS is True)
- RBMQ_USERNAME: rabbitmq regular account username (defaults to RBMQ_ADMIN_USERNAME)
- RBMQ_PASSWORD: rabbitmq regular account password (defaults to RBMQ_ADMIN_PASSWORD)

(Optional settings) 

- AGENTS_UL_EXCHANGE:  exchange topic to send data to agents
- AGENTS_DL_EXCHANGE:  exchange topic to receive data to agents
- CHECK_IN_QUEUE: rabbitmq queue name where agents must publish to perform check in.
- AGENT_UPDATE_QUEUE: rabbitmq queue name where high priority messages from agents are published.


helyOS settings
===============

- ENCRYPT (not implemented yet): none | agent | helyos | helyos-agent. RSA encription betwenn helyos core and agent. 
  This is an additional encription to the TSL layer used by the RAbbitMQ (default = none)
- MESSAGE_RATE_LIMIT:  maximum burst of number of messages per second that an agent is allowed publish to helyOS. (default = 150)  
- MESSAGE_UPDATE_LIMIT: maximum burst of number of database updates per second originated from messages publishing. E.g. status update messages. (default = 20)
- WAIT_AGENT_STATUS_PERIOD:  time in seconds that helyOS waits for an agent to change to the required status before triggering a mission. (default = 20)

(Optional settings)

- AGENT_REGISTRATION_TOKEN:  It is used to authenticate the agents that were not previoulsy registered in helyOS.
- MOCK_SERVICES: True or False.  If True, the services are mocked. It is used only for automated tests purposes. (default = False)
- TLS_REJECT_UNAUTHORIZED: True or False.  If True, the TLS connection is rejected if the certificate is not valid. (default = True)
- DEBUG: True or False.  If True, the helyos core log is more verbose. (default = False)


Configuration Files
-------------------
- **helyos_private.key** and **helyos_public.key**: RSA keys in PEM format. They are used to encrypt and sign helyOS messages sent to agent. They are located in `/etc/helyos/.ssl_keys/`.
- **ca_certificate.pem**: This is the Certificate Authority (CA) that signed the RabbitMQ server certificate. It is located in `/etc/helyos/.ssl_keys/`.
- **microservices.yml**: This file contains the initial configuration of the microservices. This data can be modified later in the dashboard. It is located in `/etc/helyos/config/microservices.yml`.
- **missions.yml**: It serves as a blueprint for all available missions for the agents. It instructs the helyos core on how to orchestrate the microservices for each mission. 
  This data can be modified later in the dashboard. It is located in `/etc/helyos/config/missions.yml`.

Optional Database Customization 
===============================

    Any file with the extension `*.sql` in the folder `/etc/helyos/db_initial_data/` will be executed when the database is created. 
    This can be used, in principle, to pre-populate the database with data, but also to create extra tables, views and functions  that will be automatically available in the GraphQL server.
    

Example
-------

Snippet of a *docker-compose.yml*

.. code:: 

    version: '3.5'
    services:

        database:
            container_name: helyos_database
            image: postgres:13
            ports:
                - "5432:5432"
            volumes:
                - postgres_data:/var/lib/postgresql/data/
            networks:
                - control-tower-net
        
        helyos_core:
            image: helyos_core:2
            ports:
                - 5002:5002  # websocket
                - 5000:5000  # GraphQL
                - 8080:8080  # HelyOS Dashboard
            volumes:
                - ./my_folder/yard_map_data.sql:/etc/helyos/db_initial_data/yard_map_data.sql
                - ./my_folder/microservices.yml:/etc/helyos/config/microservices.yml
                - ./my_folder/missions.yml:/etc/helyos/config/missions.yml
                - ./my_folder/helyos_private.key:/etc/helyos/.ssl_keys/helyos_private.key
                - ./my_folder/helyos_public.key:/etc/helyos/.ssl_keys/helyos_public.key
                - ./my_folder/ca_certificate.pem:/etc/helyos/.ssl_keys/ca_certificate.pem
  
            environment:
                # DATABASE
                - PGUSER=postgres
                - PGDRIVER=QPSQL
                - PGPASSWORD=${PG_PASSWORD}
                - PGHOST=helyos_database
                - PGDATABASE=my_application_db
                - PGPORT=5432
                - 
                # RABBITMQ
                - RABBITMQHOST=rabbitmq.server.com
                - RABBITMQPORT=5672
                - RBMQ_API_PORT=15672  
                - RBMQ_SSL= False 
                - RBMQ_API_SSL= False
        
                # RBMQ ACCOUNTS
                - CREATE_RBMQ_ACCOUNTS=True #if helyOS creates the rabbitmq accounts 
                - RBMQ_ADMIN_USERNAME=helyos_core 
                - RBMQ_ADMIN_PASSWORD=${RBMQ_PASSWORD} 
    
                # GRAPHQL 
                - GQLPORT=5000
                - JWT_SECRET=${MY_SECRET_KEY}
            networks:
                - control-tower-net
                
            depends_on:
                - database

To run use the command: ``docker-compose up``.


