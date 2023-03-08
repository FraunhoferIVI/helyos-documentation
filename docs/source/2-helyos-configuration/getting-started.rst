Getting Started 
===============
The easiest way to run helyOS locally is by using the docker image. The docker image can run locally or be deployed in a cloud provider.  
One can run the docker image using a single line command: 

.. code:: 

    docker run helyos_core -p 5000:5000 -e  PGUSER=helyos_admin  -e  PGHOST=database.io  -e  … 

However, it is more convenient to declare all the components and environment variables in the *docker-compose.yml*.

The helyOS environment variables are:

- PGUSER: username for postgres.
- PGPASSWORD: password for postgres.
- PGDATABASE: database name of your application. A postgres instance can host multiple databases.
- PGHOST: hostname for postgres.
- PGPORT: connecting port of the postgres.
- GQLPORT: server port for user interface
- ENCRYPT: how is encryption handled: none | agent-pubkey 
- RBMQ_HOST: hostname for rabbitmq server.
- RBMQ_PORT: connecting port of the rabbitmq server.
- RBMQ_API_PORT:  configuration port api port for rabbitmq server.      
- CREATE_RBMQ_ACCOUNTS: True or False.  helyOS automatically creates the rabbitmq accounts 
- RBMQ_ADMIN_USERNAME: rabbitmq admin username (required if CREATE_RBMQ_ACCOUNTS is True)
- RBMQ_ADMIN_PASSWORD: rabbitmq admin password (required if CREATE_RBMQ_ACCOUNTS is True)
- RBMQ_USERNAME: rabbitmq regular account username (defaults to RBMQ_ADMIN_USERNAME)
- RBMQ_PASSWORD: rabbitmq regular account password (defaults to RBMQ_ADMIN_PASSWORD)
- AGENTS_UL_EXCHANGE: rabbitmq exchange topic to send data to agents
- AGENTS_DL_EXCHANGE: ${AGENTS_DL_EXCHANGE}
- CHECK_IN_QUEUE: rabbitmq queue name where agents must publish to perform check in.
- AGENT_UPDATE_QUEUE: ${AGENT_UPDATE_QUEUE}   

Snippet of a *docker-compose.yml*

.. code:: 

    version: '3.5'
    services:
        database:
            image: postgres:10
            ports:
                - "5432:5432"
            volumes:
                - postgres_data:/var/lib/postgresql/data/
            networks:
                - control-tower-net
        
        helyos_core:
            image: helyos_core:1.2.0
            ports:
                - 5002:5002  # websocket
                - 5000:5000  # GraphQL
                - 8080:8080  # HelyOS Dashboard
            environment:
                # DATABASE
                - PGUSER=postgres
                - PGDRIVER=QPSQL
                - PGPASSWORD=null
                - PGHOST=database
                - PGDATABASE=my_application_db
                - PGPORT=5432
                # RABBITMQ
                - ENCRYPT=none  # none | agent | helyos | helyos-agent
                - RBMQ_HOST=rabbitmq.ivi.fraunhofer.de
                - RBMQ_PORT=5672
                - RBMQ_API_PORT=15672  
        
                # RBMQ ACCOUNTS
                - CREATE_RBMQ_ACCOUNTS=True #if helyOS creates the rabbitmq accounts 
                - RBMQ_ADMIN_USERNAME=${HELYOS_RBMQ_USERNAME} #if CREATE_RBMQ_ACCOUNTS is True
                - RBMQ_ADMIN_PASSWORD=${HELYOS_RBMQ_PASSWORD} #if CREATE_RBMQ_ACCOUNTS is True
    
                # AGENT => HELYOS
                - AGENTS_UL_EXCHANGE=${AGENTS_UL_EXCHANGE}
                - CHECK_IN_QUEUE=${CHECK_IN_QUEUE}
                - AGENT_UPDATE_QUEUE=${AGENT_UPDATE_QUEUE}    
                # HELYOS => AGENT
                - AGENTS_DL_EXCHANGE=${AGENTS_DL_EXCHANGE}
    
                # GRAPHQL 
                - GQLPORT=5000
            networks:
                - control-tower-net
                
            depends_on:
                - database

To run use the command: ``docker-compose up``.


