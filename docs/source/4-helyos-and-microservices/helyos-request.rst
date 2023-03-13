Request of Missions from App to helyOS 
--------------------------------------
Mission Request
^^^^^^^^^^^^^^^
To create a mission, the software developers must insert a row in the table of work processes. They can use the GraphQL language or the helyOS Javascript SDK for that.  

.. code::

    {	yardId: 1,
        workProcessTypeName: 'driving',
        status: 'dispatched',
        toolIds: [1],
        waitFreeAgent: true,
        data: {
            "x":-25507.5521640504,"y":20096.963720561474,
            "anchor":"front","tool_id":1,"orientation":918,
            "schedStartAt":"2021-11-01T15:09:49.802Z",
        }
    }

Example of mission: The mission type is “driving” and it employs the agent (tool) with id=1.

- **The follow fields are processed by helyOS core:**

  - **yardId:** Database id of yard.
  - **workProcessTypeName:** One of the mission names previously defined in the helyOS dashboard (Define Missions view).
  - **status:**  'draft' | "cancelling" |  'canceled' | 'dispatched' | "preparing resources" | "calculating" | "executing" |  "succeeded". When creating, you can only define as 'draft' or "dispatched". Once the status is set as 'dispatching', the helyOS will prompt the execution of the mission.  When updating, you can only set the status as "cancelling" or "dispatched".
  - **toolIds:** A list containing only the database ids of the agents taking part in the mission. This agents will be reserved by helyOS core.
  - **waitFreeAgent (optional):** Default is true. It defines if helyOS must wait all agents listed in toolIds to report the status free before triggering the mission calculations.  Set false if you don't need to reserve the agent and you can pile up assignments in the agent queue. Notice that this may produce assignments calculated with outdated yard data. 

- **The follow field is saved by helyOS core and just forwarded to the microservice(s).**

  - **data:** The Mission Data, a user-defined JSON specific of the application.  This field will be forward to the microservices. The microservice will received the Mission Data from the client software and the yard state from helyOS core (HelyOSContext). The developer must therefore add here any necessary information that is not present in the yard state. For example, pointing the agent Id that will receive the assignment, or the ordering that the assignment must be dispatched through multiple agents.  

Request of assignments from helyOS to Microservices
---------------------------------------------------
Once the mission is triggered, the helyOS will dispatch HTTP requests to the related microservices. These requests containing the mission data from the app, as defined previously, and the context. 

.. figure:: ./img/context_example.png
  :align: center
  :width: 500

  Context example

The **context** contains all the yard information at the moment of the dispatch, and calculations results from previous steps from other microservices in **context.dependencies**.

.. figure:: ./img/helyos_context_example.png
  :align: center
  :width: 600

  helyOSContext example

.. note:: 
  Note: map objects can identify any entity existing in the physical space: obstacles, gates, lanes, parking spots, fields, etc.

Assignment Creation
^^^^^^^^^^^^^^^^^^^
Assignments are created by microservices in the *Assignment Planner* domain. A microservice can create one or more assignments per mission, and can define the dispatch order to agents.

.. figure:: ./img/assignment_example.png
  :align: center
  :width: 600

  Assignment example

Microservice response data structure as defined in the Assignment planner API.

- **request_id:** service generated job id.
- **status:** "failed" | "pending" | "successful".
- **results:** it is an array of assignments where each assignment is ascribed to a tool id (agent). 
- **dispatch_order:**  When assignment must be executed sequentially, this variable is defined as an array of the element indexes of the results array. The order of the indexes defines the order that the correspondent assignment will be dispatched to the agent.

.. note:: 
  | Note: You cannot send more than one mission at once to a same agent. However, you can SEND SEVERAL ASSIGNMENTS to a same agent! For this, add the assignments as **results** items with the same **toold_id**.
  
  | Use the **dispatch_order** field to let helyOS to sequentially dispatch the assignments to a same agent. Otherwise the assignments will be sent simultaneously; in this case, the agent would need to be smart enough to consume and handle the assignments in the correct order.


Mission Sequence
^^^^^^^^^^^^^^^^
The following figure illustrates the process of mission request from the point of view of the Client application.

1. The client logs to helyOS and receives an authentication token, which will be used for subsequent requests.
2. Client makes the mission request and the helyOS core reserves all agents necessary for that mission. 
3. The helyOS calls the microservices to calculate the assignment data for the requested mission (which microservices and their order is pre-configured for each mission type).
4. helyOS receives the assignment data from the microservice and distributes them to the agents using rabbitMQ.
5. When the agent finishes the assignments, they will inform helyOS. helyOS may release the agent (reserved = False)

.. figure:: ./img/mission_creation.png
  :align: center
  :width: 600

  The process of mission creation from client











