Getting Started 
===============
The easiest way to run helyOS locally is by using the docker image. The docker image can run locally or be deployed in a cloud provider.  
The docker image can be run by using a single line command: 

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
- ENCRYPT: how is the application-level encryption handled: none | agent-pubkey 
- RBMQ_HOST: hostname for rabbitMQ server.
- RBMQ_PORT: connecting port of the rabbitMQ server. (usually 5671 for secure and 5672 for not-secure connections)
- RBMQ_API_PORT: configuration port of rabbitMQ API. (usually 15671 for secure and 15672 for not-secure connections)     
- RBMQ_SSL:  Use TLS protocol for encrypted communication to send and receive messages.
- RBMQ_API_SSL:  Use TLS protocol for encrypted communication to configure RabbitMQ.
- CREATE_RBMQ_ACCOUNTS: True or False.  helyOS automatically creates the rabbitMQ accounts 
- RBMQ_ADMIN_USERNAME: rabbitMQ admin username (required if CREATE_RBMQ_ACCOUNTS is True)
- RBMQ_ADMIN_PASSWORD: rabbitMQ admin password (required if CREATE_RBMQ_ACCOUNTS is True)

Advanced parameters (optional):

- RBMQ_USERNAME: rabbitMQ regular account username (defaults to RBMQ_ADMIN_USERNAME)
- RBMQ_PASSWORD: rabbitMQ regular account password (defaults to RBMQ_ADMIN_PASSWORD)
- AGENTS_UL_EXCHANGE: rabbitMQ exchange topic to send data to agents.  (default = xchange_helyos.agents.ul)
- AGENTS_DL_EXCHANGE: rabbitMQ exchange topic to receive data from agents. (default = xchange_helyos.agents.ul)
- ANONYMOUS_EXCHANGE: rabbitMQ exchange topic to receive data from not yet registered agents  (default = 'xchange_helyos.agents.anonymous')
- CHECK_IN_QUEUE: rabbitMQ queue name where helyOS will received the agents check-in data. (default = 'agent_checkin_queue')
- AGENT_UPDATE_QUEUE: rabbitMQ queue name where helyOS will received the agents update data. (default = 'agent_update_queue')
- AGENT_STATE_QUEUE: rabbitMQ queue name where helyOS will received the agents update data. (default = 'agent_state_queue') 

You don't need to set the variables for the advanced parameters unless you have a unique situation that requires specific settings for RabbitMQ.


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
                - RBMQ_HOST=rabbitmq.server.com
                - RBMQ_PORT=5672
                - RBMQ_API_PORT=15672 
                - RBMQ_SSL=False # True | False (default = False) 
                - RBMQ_API_SSL=False # True | False (default = RBMQ_SSL )  
        
                # RBMQ ACCOUNTS
                - CREATE_RBMQ_ACCOUNTS=True # helyOS automatically creates the rabbitmq accounts (default = True)
                - RBMQ_ADMIN_USERNAME=${HELYOS_RBMQ_USERNAME} # if CREATE_RBMQ_ACCOUNTS is True
                - RBMQ_ADMIN_PASSWORD=${HELYOS_RBMQ_PASSWORD} # if CREATE_RBMQ_ACCOUNTS is True
    
                # AGENT => HELYOS
                - AGENTS_UL_EXCHANGE=${AGENTS_UL_EXCHANGE}

                # HELYOS => AGENT
                - AGENTS_DL_EXCHANGE=${AGENTS_DL_EXCHANGE} 
    
                # GRAPHQL 
                - GQLPORT=5000

            volumes:
                - ./certificates/ca_certificate.pem:/.ssl_keys/ca_certificate.pem:ro # if RBMQ_API_SSL is True.
            networks:
                - control-tower-net
                
            depends_on:
                - database


helyOS must access the RabbitMQ server CA certificate to connect RabbitMQ with TLS protocol.
For this, map or copy the certificate to the path: `/.ssl_keys/ca_certificate.pem`.

To run use the command: ``docker-compose up``.


