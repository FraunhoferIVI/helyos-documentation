helyOS and Agents
=======================================
The communication between helyOS and agents is intermediated by the message broker rabbitMQ. Most of the messages are sent and received using topic exchanges. 
Before helyOS send assignments to the agent, the identification number (uuid) of the agent must be registered in the helyOS database. This can be done via dashboard 
or via a direct graphQL request, or, in special cases, automatically during the “check-in” by using a pre-defined uuid as token.
 
.. toctree:: 
    :maxdepth: 1

    agents-and-helyos