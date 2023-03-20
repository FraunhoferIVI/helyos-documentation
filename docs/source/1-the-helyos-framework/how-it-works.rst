How It works
============

Software Components
-------------------

The helyOS framework contains three software components: 

- **helyOS core**: a flexible backend software that receives mission requests from frontend apps and uses microservices to transform these requests into robot assignments. 
- **helyOS Agent SDK**: a Python library used to connect agents (robots and vehicles) to helyOS core via rabbitMQ.  
- **helyOS JavaScript SDK**: a JavaScript library used to create frontend apps that communicate with helyOS core. 

Each application built within the helyOS framework is shaped by connecting helyOS with function-specific microservices, e.g., routing, trajectory planning, map update, swarm behavior, etc.

Control Strategy
----------------

The helyOS framework embraces the principle of separation of concerns, featuring centralized command but distributed processes. It adopts a microservice architecture where expert subsystems are independently developed according to predefined domains. This approach complies with the use cases found in yard automation and smart farming and it is in line with the most modern practices in system integration. 
 
In contrast to the traditional "leader-follower" approach, helyOS core does not micromanage assignments. It distributes the assignments including all necessary data upfront to the agent, who is part of the autonomous domain of the application. This allows the agent maximum independence while still retaining the possibility of exchanging data with expert systems inside the autonomous domain. This data exchange can be used, for example, in real-time corrections commands to support a running assignment. This approach favors the gradual development of more autonomous agents. 

.. note:: 
    Note that agents can also request missions for themselves from helyOS core. This feature can be exploited when the resolution of a given assignment is not possible inside the autonomous domain, for example, to overcome deadlocks or to emulate a leader-follower approach.

 
Data Formats
------------
helyOS uses JSON formats. Except for a minimum set of required fields used to control the data flow, developers can freely choose the data structure of the assignment, map and sensor data.  In the helyOS framework, most of the data formats are resulted from agreements between user interface programmers, mission planner developers and the robot controller developers. 

Communication
-------------

helyOS uses the HTTP protocol to communicate with microservices and frontends, and RabbitMQ to communicate with the agents. helyOS uses RabbitMQ and consequently the AMQP protocol. AMQP originated from the financial industry, where it is used for the safe communication of financial data. Its main developers are Cisco, Red Hat, IONA and Twist. This protocol allows both *produce/consume* queue and *publish/subscribe* patterns with advanced routing features. RabbitMQ is also a reference for microservice architecture and the development of remote procedure calls.
