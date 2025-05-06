# Proposed timeline

## Week 1: 5/5 - 5/11
### Deliverables:
| Component       | Implemented | Tested |
|-----------------|-------------|--------|
| client service  | [ ]          | [ ]     |
| REST service    | [ ]          | [ ]     |
| Redis instance  | [ ]          | [ ]     |
| Task dispatcher | [ ]          | [ ]     |
| local worker    | [ ]          | [ ]     |

A component will be considered to be fully "implemented" when it properly working and incorporates all of the needed requirements outlined below.

A component will be considered to be fully "tested" when appropriate unit and integration tests have been written that all pass successfully.


#### Client service

- utilizes the REST service to pass serialized functions to the FaaS infrastructure

- serializes the body of the function, and passes the function to the FastAPI endpoint /reigster_function

- will receive the UUID of the function once it has been successfully registered

- will be able to properly display the result of a function to the client

#### REST service

- constructed using FastAPI

- receives serialized functions via the /register_function endpoint

- conveys these serialized functions to Redis

- receives a UUID from Redis and returns this back to the client

- should have a /status interface that can retreive the status for a given task UUID

- should have a /results interface that can receive the results for a given task UUID and return that serialized object to the client

#### Redis instance

- stores all functions in key/value pairs from the REST service

- will use the UUIDs to represent the keys
    
- will use JSON objects for the values
    - e.g. { "body": f"{function_body}, "input_arguments": f"{fn_arguments}, "status": f"{status}", "results": f"{results}" }

- the state of the task will be stored in the database. Tasks will progress through the following lifecycle:
    - QUEUED: task is created and stored in the database
    - RUNNING: task has been sent to a worker
    - COMPLETE: task has been completed successfully
    - FAILED: task has completed and ran unsuccessfully

- all communication with Redis should come either from the REST service, or the task dispatcher

#### Task dispatcher

- will take tasks from Redis and launch them on the local worker
    - does so in a completely separate component from the FastAPI and Redis components

- will be configured to listen on the Redis tasks queue `Redis.sub("tasks")`, which notifies the task scheduler once new tasks are registered

- when a task is received, it will retreive the function body and input arguments from Redis and dispatch them for execution on the local worker
    - when it is complete, it will write back to Redis

#### Local worker

- receives the task_id, the serialized function body, the serialized parameters

- returns the task_id, the status of the function and the serialized result of the function to the task dispatch

- NOTE: all workers (not just the local worker) should implement a multiprocessing pool

##### Desired final deliverable

We will have a working infrastructure that will:

1. use the client script to serialize a function and pass it to the REST service
2. The REST service will properly register function and return the UUID of the successfully-registered function to the client
3. the client will then be able to successfully invoke the function via the REST service
4. The task scheduler will learn that the function has been invoked via the Redis queue
5. The task scheduler will then run the function via their local worker and return the outcome to Redis
6. The client will be able to obtain the status and results of the client via the REST service

## Week 2: 5/12 - 5/18

### Deliverables
| Component                     | Status | Tested |
|-------------------------------|--------|--------|
| Pull worker                   | [ ]     | [ ]     |
| Push worker                   | [ ]     | [ ]     |
| Task scheduler support - pull | [ ]     | [ ]     |
| Task scheduler support - push | [ ]     | [ ]     |
| Fault tolerance               | [ ]     | [ ]     |

#### Pull worker

- will, in a loop:
    1. Poll the task scheduler for tasks
    2. execute the task received from the task scheduler (if applicable)
    3. return the result of the task to the task scheduler (if applicable)

#### Push worker

- will leverage ZMQ's DEALER/ROUTER pattern to push workloads to the worker

- once received, the worker will process the task, and return the result to the task scheduler

- a method to properly load balance work amongst all push workers should be implemented

#### Task scheduler support - pull worker

- will implement a ZMQ REQ/REP queue to pass tasks to workers
    - the task scheduler will listen in a REP loop, and workers will request tasks from the scheduler when they are available

#### Task scheduler support - push worker

- implement a ZMQ DEALER/ROUTER pattern to send tasks to waiting workers
    - these tasks should be pushed to workers as soon as they are received

#### Fault tolerance

- the system must be robust to failures
    e.g. serialization errors or errors with the function
        - appropriate errors and error codes should be returned to the client

- the system must also be robust to failures in worker nodes
    - workload allocated to dead workers must have a mechanism to be re-allocated to live workers in the event workers fail


##### Desired final deliverable
- at this point, all of the core functionality of the project should be complete.
    - we will be able to pass functions to both push and pull workers and have them be processed appropriately

- our system will be able to detect failures in push and pull workers, and recover from them appropriately
    - work is able to be rerouted to other workers as needed

## Week 3: 5/19 - 5/23

| Component                | Status |
|--------------------------|--------|
| Finalized testing        | [ ]     |
| Finalized error handling | [ ]     |
| Performance Report       | [ ]     |
| Technical Report         | [ ]     |
| Testing Report           | [ ]     | 

## Finalized testing and error handling

- at this point in the project, we expect to have a fully functioning prototype incorporating all necessary features

- building in time to catch up and fill out any additional testing and error handling we catch or failed to write up to this point

## Performance Report

- we will use the client to evaluate performance of both our push and pull workers
    - the local worker on the task scheduler can serve as a baseline for both

- what dimensions should we evaluate and how can we quantify the different aspects of system performance?
    - latency vs. throuhput

- what types of functions should we use?
    - no-op
    - sleep tasks
    - calculation intensive

- weak scaling study
    - increase the amount of work as you increase the number of workers/processes
        1 worker == 5 tasks, 4 workers == 20 tasks, 8 workers == 40 tasks

- there should be more than one worker process per worker

## Technical Report

- a comprehensive view of how we implemented our solution

- any important decisions (preferably documented in the Design decision tracker)

- and rough issues (preferably documented in the issue tracker)

## Testing Report

- we will be provided a set of integration tests that will be used to test our project

- we should also implement our own testing suite on top of it

- in addition to the actual tests, a report on our test cases, any other testing done, and the limitations of our testing will need to be written

# Issue Tracker

# Design decision tracker

# RAID Log

## R: Collaboration
- right now, we both have separate git repos for the client
    - how can we ensure we are both working on the same version of the project?
        - how do we want 

## R: Dividing work
- many parts are dependent on the other
    - how can we successfully divide work so both of us are working effectively early on in the project?