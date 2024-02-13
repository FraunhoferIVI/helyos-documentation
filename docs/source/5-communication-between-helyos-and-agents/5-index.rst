.. _helyos-and-agents:

helyOS and Agents
=======================================
Communication between helyOS and agents is conveyed by the RabbitMQ message broker. Most of the messages are sent and received using topic exchanges. The agent identification number (uuid) must be registered in the helyOS database. This can be done via dashboard or via a direct GraphQL request, or, in special cases, automatically during the “check-in” process via a registration token.
 
.. toctree:: 
    :maxdepth: 1

    agents-and-helyos