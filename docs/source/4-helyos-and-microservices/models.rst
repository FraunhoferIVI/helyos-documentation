

.. _models_description:

Models Description
------------------


  .. code-block:: typescript
      :caption: Requesta data send from helyOS to the microservice.

      HelyOSContext  {
          agents: AgentModel[]; // arrray of agents relevant to the mission.

          map: {
                id: number, 
                origin: { lat: number, long: number}
                map_objects : MapObjectsModel[]; // array of all map objects in the yard.
                };


          orchestation: {
                          current_step: string, // name of the current step in the mission.
                          next_step: string,    // name of the next step in the mission.
                        };

          dependencies: { step: string, 
                          requestUid: string, 
                          response: any 
                        }[]; // array of data responses from microservice of previous steps.
                        
      }


  .. code-block:: typescript
      :caption: Agent and map object models.

      AgentModel {
          id: number;
          uuid: string;
          name: string;
          agent_type: string;
          agent_class: "vehicle" | "assistant" | "tool" | "charge_station";
          pose: { x: number, y: number, z:number,  orientations: number[] };
          status: "free" | "busy" | "ready" | "not_automatable";
          connection_status: "off-line" | "on-line";
          public_key: string;

          data_format: string;
          resources: any;
          geometry: any
          sensors: any;  

          interconnections: {uuid: string, agent_type: string}[];
      }

      MapObjectsModel {
          id: number;
          name: string;
          type: string;
          data_format: string;
          data: any;
          metadata: any;
      }




      
  In general the microservice response is a JSON object with the following structure:

  .. code-block:: typescript
      :caption: Response data structure as defined in the Assignment planner API.

      HelyOSMicroserviceResponse {
          request_id?: string;  // generated job id. Can be used to poll results from long running jobs.

          status: "failed" | "pending" | "successful";  .

          results: AssignmentPlan[] | MapUpdate | any;

          dispatch_order?: number[][]; 

          orchestration?: {
                    nex_step_request: [step: string]: any; // input data to be sent to the next microservice(s).
                    }
      }


      
