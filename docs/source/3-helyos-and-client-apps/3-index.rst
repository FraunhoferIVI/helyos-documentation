helyOS and Client Apps
======================
helyOS core responds to Postgres database events. That is, the creation, update or delete of rows in the database tables trigger actions inside the helyOS core. 
Therefore the Client Applications communicate with helyOS core by interacting with the helyOS database. This interaction uses the GraphQL language.

.. toctree:: 
    :maxdepth: 1

    application-accounts
    communication