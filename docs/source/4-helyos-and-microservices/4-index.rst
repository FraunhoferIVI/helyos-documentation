helyOS and Microservices
========================

The communication between helyOS and microservices is configured in the helyOS dashboard. Every time helyOS receives a mission request, it makes calls to the relevant microservices. If the microservice results are not immediately available, helyOS will poll the results in subsequent periodic calls. The microservices must serve endpoints according to the helyOS definitions for microservices APIs.

.. toctree:: 
    :maxdepth: 1

    helyos-request
    models