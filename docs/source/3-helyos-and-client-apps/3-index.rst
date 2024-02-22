helyOS and Client Apps
======================
helyOS core responds to Postgres database events. This means that the creation, update or deletion of rows in the database trigger actions inside helyOS core. Therefore, the client applications communicate with helyOS core by interacting with the helyOS database. This interaction uses the GraphQL language.

.. toctree:: 
    :maxdepth: 1

    application-accounts
    communication
    request-missions