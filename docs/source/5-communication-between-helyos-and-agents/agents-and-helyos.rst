Overview
--------

.. figure:: ./img/agent_receving_mission.png
    :align: center
    :width: 600

    The process of agents receiving mission assignments


Only if the agent UUID is registered in helyOS database, it can exchange messages with helyOS. The messages are used to inform the agent status and perform the assignments. 
Usually the change from status "non automatable" to "free" must be done manually by the driver. 

| Note that before receiving any assignment, the agent is reserved for the mission. That is, the agent changes the status from "free" to "ready" (i.e., read for the mission) upon helyOS Reserve request. Once the agent finishes the assignment, the agent will not set itself as "free", but as "ready". This is because helyOS may sent him a second assignment belonging to the same mission. For this reason, the agent must wait the "Release" signal from helyOS to set itself "free".|


Exchange, routing-keys and queues in rabbitMQ
---------------------------------------------

.. figure:: ./img/helyOS_rabbitMQ.png
    :align: center
    :width: 600

    helyOS and rabbitMQ

All the messages exchanged between helyOS and agents have the common fields:

- **type:** string, ex: "checkin", "assignment", "cancel", etc..
- **uuid:** string,
- **body:** JSON object.

Check in agent in helyOS
------------------------
To receive assignments from helyOS, the agent must perform a procedure called "check in".

In the check in procedure the agent will 

- Connect as anonymous [*]_ to rabbitMQ and send its identification data using the routing key **agent-{uuid}-checkin**.
- Receives an individual username and password to access rabbitMQ.
- Create queues to receive the messages using the routing key to **agent-{uuid}-assignments** and **agent-{uuid}-instantActions**.

.. [*] Agents logged as anonymous are not allowed to perform any other operation than “check in”.

.. figure:: ./img/agent_check_in.png
    :align: center
    :width: 600

    Agent check in example

Check in data sent by the agent to helyOS.

- **Type** = "checkin".
- **Geometry:** JSON informing the physical geometry data of the vehicle.
- **Yard_uid:** Unique identifier of the yard as registered in the dashboard.

helyOS will respond with the following data:

.. figure:: ./img/agent_check_in_response.png
    :align: center
    :width: 600

    Agent check in response

Check in response sent by helyOS to the agent.

- **Type** = "check in".
- **map:** JSON with the map information from yard.
- **Rbmq_username:** rabbitMQ account to be used by this agent.
- **Rbmq_password:** rabbitMQ account to be used by this agent.
- **Password_encrypted:** If true, the rbmq_password field is encrypted with the agent public key.

Check in using python code:

.. code:: python

    def checkin_pseudo_code():
        # step 1 - connect anonymously
        temp_connection = connect_rabbitmq(rbmq_host, "anonymous", "anonymous")
        gest_channel = temp_connection.channel()

        # step 2 - create a temporary queue to receive the checkin response
        checkin_response_queue = gest_channel.queue_declare(queue="")

        # step 3 - publish the check in request
        uuid = "y4df7293-5aab-46e2-bf6b"
        publish_in_checkin_exchange_topic(yard_id=1, 
                                        uuid: uuid,
                            routing_key: f"agent-{uid}-checkin,
                                        status="free",
                                        agent_metadata=data,
                                        reply_to= checkin_response_queue)    

        # step 4 - start consume checkin_response_queue and get authentication data
        username, password = listen_checkin_response(checkin_response_queue)

        # step 5 - log to rabbitmq as agent with full permission rights.
        helyos_checked_in_connection = connect_rabbitmq(rbmq_host, username, password)

The same code using helyOS-agent-sdk python package:

.. code:: python

    from helyos_agent_sdk import HelyOSClient, AgentConnector
    helyOS_client = HelyOSClient(rbmq_host,rbmq_port,uuid="y4df7293-5aab-46e2-bf6b")
    helyOS_client.perform_checkin(yard_uid='1', agent_data=data, status="free")
    helyOS_client.get_checkin_result()

    helyos_checked_in_connection = heylOS_client.connection

helyOS-agent-sdk HelyOSClient and AgentConnector have many other attributes and methods to send and receive data from helyOS core in the correct data format. 
Check the documentation at https://fraunhoferivi.github.io/helyOS-agent-sdk/build/html/index.html.

helyOS reserves agent for mission
---------------------------------
When helyOS needs an agent to take part in a mission, helyOS core will reserve this agent before sending it assignments. This is done via the instant action routing key, *agent.{uiid}.InstantAction* . helyOS requests the agent to be in **"ready"** status (status="ready" and reserved=True). After the assignment is finished, the agent updated its status to **"free"**.  After the assignment is complete, helyOS will release or not the agent depending on the existence of further assignments in that mission. The release message is also delivered via instant actions.

HelyOS sends assignment to agent
--------------------------------
helyOS will send an assignment to the agent **only if the agent status is "ready"**.   This is done via the routing key *agent.{uiid}.assignments*. 

.. figure:: ./img/assignment-data-format.png
    :align: center
    :width: 700

    Assignment object data format

An easy-to-implement security mechanism is to check the identity of the assignment sender. This is an embedded feature from RabbitMQ. For example, if your agent should only execute assignments from helyOS core, you can filter assignments originated from the RabbitMQ account "helyOS_core".

Agent requests a mission 
------------------------

Not only client apps but also agents can request missions to helyOS core. This features is useful for situations as the following:

- A smart camera identify a new obstacle and requests a mission to update helyOS map by sending the position of a new obstacle.
- A tractor requests a mission to ask assistance of another agent to execute a task.
- A truck find itself obstructed by a fixed obstacle, it requests a mission to helyOS to remove itself from this deadlock situation.



