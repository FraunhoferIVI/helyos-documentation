How to Configure helyOS Core as Backend
=======================================
In general terms, to configure helyOS as the backend of a yard automation application, the developer needs to access the helyOS dashboard and perform the following tasks:

1. Connect helyOS to a running rabbitMQ server.
2. Register the frontend apps in the "Register App" dashboard tab. 
3. Register the yard in the “Yards” dashboard tab. A yard represents the fenced area containing the autonomous agents (vehicle or robots).
4. Register the agents (vehicles or robots) in the "Register Agents" dashboard tab. 
5. Define the missions available for your application in the dashboard “Define Missions” menu. Examples of missions. *"drive_from_a_to_b"*, *"seed_field"*, etc.
6. Register microservices in the "Microservices" view. They will be used to process the mission's request data and produce assignment data, e.g. a trajectory path for the vehicle.
7. Create "Mission Recipes", that is, associate each mission to one or more of the registered microservices.




  Using helyOS core as a backend, the front-end developer can run the user interface (e.g. Graphana boards, web applications, etc.). The frontend developer may use either **helyOS JavaScript SDK**  (for web applications) or the **GraphQL** language (for any kind of application) to create missions and access all data from the yard and the automated vehicles.
