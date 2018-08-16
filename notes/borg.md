# Borg

#### What is Borg?
  - Cluster management system
  - Schedules, starts, restarts, handles monitoring for full range of applications at Google
#### What are the benefits of Borg?
  - Hides the details of resource management and failure handling so that users can develop apps
  - Operates with very high reliability and availability
  - Scalability: Runs workloads across tens of thousands of machines

### The User Perspective
##### Who are the Users of Borg?
  - Google Developers
  - SREs
- How do they operate Borg?
  - Users submit their tasks as jobs to Borg, which internally runs inside a Borg Cell

#####  The workload
  - What are the two types of workloads which are run on Borg? Describe with examples.
    - Long running services that serve short lived latency sensitive requests
      They need to be served within few microseconds to a few hundred milliseconds
      Used to serve API calls for GMail, Google Docs and BigTable
    - Batch Jobs
      Not very sensitive to short term performance fluctuations
  - Give examples of Google's stuff that runs on Borg
    - MapReduce, GMail, Google Docs, BigTable, CFS
  - What are the two job priorities described in the paper?
    - High Priority - Production Jobs
    - Low Priority - Non Production Jobs
  - What jobs are prod? and what jobs are non-prod?
    - High Priority - Production Jobs
    - Low Priority - Non Production Jobs

#####  Clusters and cells
  - What is a cell?
    - Cell is a collection of machines
    - It belongs to a cluster
    - Median Size: 10k machines
    - Machines in a cell are heterogeneous in terms of CPU, memory, disk, network,
      processor type, performance, capabilities such as external IP, flash storage
  - What is a cluster?
    - A cluster is an abstraction that represents a collection of server mahcines
    - It lives inside a building
    - It usually has one large cell + smaller scale test / special purpose cells
  - What is a data center?
    - Collection of multiple clusters
  - What is a site?
    - Collection of buildings
  - What is the average size of a Borg cell?
    - 10k machines
  - How does Borg isolate users from the above complexity?
    - Determines where in the cell to run the tasks
    - Allocating resources and installing their programs along with dependencies
    - Starting their programs
    - Monitoring the programs of the allocated jobs
    - Restart if they have failed

  ### Jobs and Tasks 
  - What is a job?
    - Represents an abstraction of an event to perform
    - Has properties: name, owner and number of tasks
    - A job has many tasks
    - Can enforce constraints - processor architecture, os version, external ip address
    - Runs in one cell
    - Possible Example: Deploy the service on 64 bit Ubuntu 16.04.01 Kernel Version: xyz with external ip address
  - What are the properties needed for a job?
    - Has properties: name, owner and number of tasks
    - Can enforce constraints - processor architecture, os version, external ip address
  - What is a task?
    - Maps to a set of Linux processes running in a container on a machine
    - Has properties: Resource Requirements and Index within the Job
    - One can specify CPU, RAM, disk space, disk access rate to a particular task
  - What are the properties needed for a task?
    - Resource Requirements and Index within the Job
    - One can specify CPU, RAM, disk space, disk access rate to a particular task
  - How do users communicate with Borg?
    - Users issue RPC to Borg for interaction
    - They write the Job descriptions BCL - Borg Configuration Language
    - Users can change the properties of some or all of the running tasks in a job by
      publishing a new job configuration to Borg
    - Tasks can be notified via SIGTERM before they are preempted by SIGKILL
  - How are job configurations written?
    - Job descriptions are written in BCL - Borg Configuration Language
  - How does BCL allow users to adjust configurations adjusted by environments?
    - Users can change the properties of some or all of the running tasks in a job by
      publishing a new job configuration to Borg
  - What are the states that the jobs and tasks go through their lifetime?
    - Figure #2 => State diagram of jobs in Borg
    - Submitted -> Pending -> Running -> Dead / Complete
  - How does the user update job specifications?
    - Users can change the properties of some or all of the running tasks in a job by
      publishing a new job configuration to Borg
  - How can a task be notified before it is killed?
    - Tasks can be notified via SIGTERM before they are preempted by SIGKILL

  ##### Allocs
  - What is an alloc?
    - Reserved set of resources on a machine on which a given set of tasks can be run
    - Resources remain assigned irrespective of whether they are used or not
    - Jobs can be submitted to be run on an alloc
  - Describe the three use-cases for Allocs?
    - Set resources for future tasks
    - Retain resources between stopping a task and starting it again
    - Gather tasks from different jobs onto the same machine
  - What is an alloc-set?
    - Group of allocs that reserve resources on multiple machines

  ##### Priority, Quota and Admission Control
  - When are priority and quota necessary?
    - When more work shows up than can be accomodated, priority and quota comes into picture
  - What is priority? How is it expressed?
    - Priority is the relative importance for jobs that are running or waiting to run in a cell
    - Every job has a priority which is expressed as a small integer value
    - High priority ones obtain resources at the expense of low priority jobs
      Could also kill low priority jobs
  - What are priority bands?
    - Four priority bands - monitoring, production, batch and best-effort
    - Prod jobs are in monitoring and production bands
  - What are preemption cascades? How would they happen? And what would you do to resolve them?
    - Pre-emption - Preventing the execution
    - Pre-emption cascades - One task preventing the execution of another task, which could
      prevent the execution of another task
    - Rescheduling occurs if high priority task bumps off a lower priority one
    - Solution: A production priority task cannot bump another production priority task
  - What is quota? Why is it important?
    - Quota is the maximum amount of resources that the user job requests can ask for at a
      given time
    - It is used to decide which jobs to admit for scheduling
    - Given a project to a team and given a quota, they cannot submit a job which demands more quota
  - How is quota expressed?
    - Quota is expressed as a vector of resource quantities (CPU, RAM, disk) at a given priority,
      for a period of time

  ##### Naming and Monitoring
  - Why is naming of jobs / tasks important?
    - The service's clients and other systems need to find the created tasks
    - Hence there needs to be a standardised name
  - How does naming work in Borg?
    - A task has a Borg Name Service name which includes cell name, job name and task number
    - This also forms the basis for the DNS name - 50.jfoo.ubar.cc.borg.google.com
    - Borg writes this information in Chubby along with the health information, so that whenever it
      changes, load balancers know where to route it to
  - How is the BNS name structured?
    - A task has a Borg Name Service name which includes cell name, job name and task number
    - This also forms the basis for the DNS name - 50.jfoo.ubar.cc.borg.google.com
  - How is health of the systems monitored?
    - Every task run under Borg contains a built-in HTTP Server which publishes information about
      health of the task and thousands of performance metrics
    - Borg monitors the health-check URL and restarts tasks that do not respond promptly or return
      an HTTP error code
    - Users may use Sigma which provides a state of all the jobs
  - Explain the capabilities of Sigma?
    - Sigma is a web-based user-interface through which users can examine:
      - The state of all the jobs
      - A particular cell
      - Drill down to individual jobs
      and examine:
      - User behavior
      - Detailed Logs
      - Execution History
  - How are voluminous logs handled?
    - Rotation
  - What is the database behind the entire monitoring infrastructure?
    - Infrastore - Scalable read-only data store with SQL like interface via Dremel

##### Borg Architecture
  - Describe the components used in Borg
    - The architecture contains a BorgMaster which is a centralised controller
    - And an agent process called Borglet which runs on each machine in a cell

  BorgMaster
  - What are the two processes that Borgmaster consists of?
    - Main BorgMaster process
    - Separate scheduler
  - What are the responsibility of BorgMaster?
    - A cell has 5 BorgMasters
    - It provides RPC APIs which perform CRUD on jobs and tasks
    - Manages state-machines for all the objects in the system - jobs, machines, tasks, allocs
    - Exposes a backup UI to Sigma
  - Describe the architecture of BorgMaster
    - Logically a single process but is actually replicated five times
    - Each replica maintains an in-memory copy of most of the state of the cell
    - This state is also recorded in a highly-available distributed Paxos based store
    - A single elected master (1 BorgMaster among 5) serves as a Paxos Leader and state mutator.
      Following are some tasks that it handles:
      - Submitting a job
      - Terminating a task
    - Master is elected in two conditions:
      - Whenever the cell is brought up
      - Whenever the elected master fails
    - It acquires a chubby lock so that other systems can find it
    - Electing a master takes ~10s, but may take more than that
    - When a replica recovers from an outage, it synchronizes its state with others
  - Let's say that there is a defect shipped in Borg, how is it debugged?
    - A periodic snapshot of Borg's state is taken which is called a *checkpoint*
    - If there is a bug shipped, checkpoints are used to restore Borg back to a particular state in
      the past, so that it can be debugged
    - Engineers try to debug the problem using a FauxMaster
    - Semantically, it means Reliable BorgMaster
    - Reads checkpoint files and contains the complete BorgMaster production code with stubbed out interfaces
    - It exposes the same RPC CRUD APIs and behaves like a real BorgMaster
    - It communicates with simulated Borglets instead of real ones so that end-user can play them one by one
      and debug problems
  - What is a checkpoint?
    - A periodic snapshot of Borg's state is called a `checkpoint` 
  - What is a FauxMaster?
    - Used for Debugging
    - Reliable BorgMaster
    - Reads checkpoint files and contains the complete BorgMaster production code with stubbed out interfaces
    - It exposes the same RPC CRUD APIs and behaves like a real BorgMaster
    - It communicates with simulated Borglets instead of real ones so that end-user can play them one by one
      and debug problems
    - What are the other uses of FauxMaster?
      - Capacity Planning
      - Sanity checks before making configuration changes

  ##### Scheduling
  - What happens when a job is submitted to Borg?
    - Borgmaster records the job persistently in Paxos store and adds the jobs' tasks to a pending queue
    - The jobs are scanned asynchronously by the `scheduler` which assigns them to machines if there are
      available resources
    - The job scanning works from high priority to low priority
  - What are the two parts in the scheduling algorithm?
    - Feasibility checking - Finding the machines on which the task would run
    - Scoring - Picks the feasible machines
  - What is feasibility checking? Explain in detail.
    - Feasibility checking is the task wherein the scheduler tries to find the set of machines that:
      - Meet the tasks constraints - Has support for external IPs etc
      - Enough available resources - Has enough CPU / RAM etc
  - What is scoring? Explain in detail.
    - In scoring, the scheduler determines the goodness of the feasible machines.
      The score takes into account the following:
      - User specified preferences
      - Minimise task re-scheduling
      - Balancing high and low priority jobs onto a single machine to allow for high priority ones to expand in
        case of load spike
  - What are the different scoring techniques the team at Google tried? What was the conclusion?
    E-PVM
    - Initially tried E-PVM for scoring
    - Given a specification of resources, E-PVM generated a single cost value
    - Advantages: Spread the load across all the machines
      Disadvantages: Left a lot of resources unutilised because it left a lot of headroom for large tasks
    Best-Fit
    - Filled in machines as tightly as possible and did not leave room in case there is a spike in load
    - Advantages: Resources are best utilised
      Disadvantages: Hurts applications with bursty loads
    Current Scoring Model
    - Hybrid - Mixture of E-PVM and Best-Fit
    - Statistically 3-5% better than best-fit
  - What are the optimisations done by the Borg Scheduler?
    - If the machine selected by feasibility checking and scheduling does not have enough available resources,
      Borg kills and reschedules the tasks that are lower in priority than the task at hand
    - The killed tasks are queued again
    - Median package installation time is 25s which includes fetching the dependencies. To overcome this, tasks
      are assigned to the machines which already contains the dependencies

  ##### Borglet
  - What is Borglet?
    - It is a local Borg agent that is present in every machine in a cell
  - What are the tasks that a Borglet performs?
    - Starts and stops tasks
    - Restarts the tasks if they fail
    - Manage local resources
    - Rolls over debug logs
    - Reports the state of the machine to other monitoring systems
  - Describe the coordination between the Borglet and the BorgMaster
    - BorgMaster polls each Borglet every few seconds to retrieve its status and send it any outstanding requests
    - Polling allows BorgMaster to control the rate of communication and prevents storms of requests on BorgMaster
    - BorgMaster prepares requests by speficying the tasks to assign to the Borglets
    - Based on the responses of Borglets, it updates the tasks which have been assigned to them
    - BorgMaster replicas have link shards which communicate with a set of Borglets
    - Borglets report the entire status but link shards report only what has changed to BorgMasters. This is nothing
      but a performance optimisation
    - If a Borglet does not respond, its machine is marked down

  ##### Scalability
  - Describe the scale Borg can operate at
    - A single Borgmaster can manage many thousands of machines in a cell
    - Several cells have above 10000 tasks per minute
    - A busy BorgMaster uses 10-14 CPU cores and upto 50 GB of RAM
  - Describe how Borg achieves this scale
    - Early versions of Borgmaster had a simple and synchronous loop that:
      - Accepted requests
      - Scheduled tasks
      - Communicated with Borglets
    - To handle larger cells, the scheduler was split into a separate process
    - Now, it operates in parallel with the other Borgmaster functions that are replicated for failure tolerance
    - A scheduler replica fetches the state changes from the BorgMaster and does a round of scheduling pass to
      assign tasks and informs the master
    - On the BorgMaster side, engineers developed separate thread pools to talk to Borglets and to serve
      read-only RPCs
  - What are the other things that make Borg scalable
    - Score Caching - Task feasibility score is cached until the properties of the task change
    - Equivalence classes - Tasks in jobs are usually of identical requirements, so Borg does feasibility check
      and scoring only once for tasks which are in one equivalence class
    - Relaxed randomization - Finding feasibility and scores for only limited machines among a large cell

##### Availability
  - What are the things that Borg does in order to increase the likelihood of completion of tasks?
    - Automatically re-scheduling tasks which are killed
    - Reduces correlated failure by spreading the tasks across different machines, racks and power modes
    - Limits the number of tasks that can simultaneously be down due to upgrades
    - Existing tasks continue to run even if BorgMaster is down
  - What are the things that Borg does to guarantee 99.99% availability?
    - Replication for machine failures
    - Admission control to avoid overload
    - Deploying instances using simple, low-level tools to minimize external dependencies

##### Utilization
  - Why does it matter?
    - Increasing utilization by a few percentage can save millions of dollars
  
  ##### Evaluation Methodology
  - What was the initial metric for utilisation and why it was inappropriate?
    - Initial metric: *average utilisation*
    - Why it didn't work - job placement constraints, rare workload spikes, heterogeneous machines
  - What is the metric for measuring utilisation which was picked after experience?
    - The metric *cell compaction* was picked after experience.
    - This meant, given a workload, how small a cell it could be fitted into
  
  ##### Cell sharing
  - What is cell-sharing? And why was it adopted?
    - Borg machines run both prod and non-prod tasks at the same time
    - Evaluated that it would require 20-30% more machines if non-prod and prod tasks were run separately
  - How was this evaluation done?

  ##### Large cells
  - Why are large cells used instead of small ones?
    - Large cells allow for large computations to be run
    - Experimented by partitioning the jobs across multiple smaller machines
    - Conclusion was - using smaller cells would require significanly more machines
  
  ### Fine-grained resource requests
  What are fine-grained resource requests?
  - Borg users request CPU in milli-cores and memeory / disk space in bytes
  Why was this designed like so?
  - Analysis was done wherein prod jobs were bucketed into buckets of fixed CPU / Memory size
  - It was concluded that it will take 30-50% more resources in this approach, hence it's resource-intensive
  
  ##### Resource reclamation
  - What is resource reclamation?
    - Most of the times, users request more resources than what their jobs need
    - In such a scenario, a good amount of resources are not utilised
    - Borg estimates the amount of resources a task will need
    - It reclaims the rest for jobs that can tolerate lower quality resources, which is called
      resource reclamation
  - What are the advantages that Google has observed out of it?
    - Better resource utilisation
    - Be able to run 20% of the workload in reclaimed resources

### Isolation
- Why is it important?
  - Machines run multiple tasks together
  - Need a good mechanism to prevent tasks from interfering each other

  ##### Security Isolation
  - How is security guaranteed?
    - chroot is used as a primary security isolation mechanism
    - External software deployed on Google App Engine and Google Compute engine are run on a VM deployed
      via Borg
  
  ##### Performance Isolation
  - Why was performance isolation needed?
    - Early versions of Borglet had primitive resource isolation environment
      - Post-hoc usage checking of memory, disk space and CPU cycles
      - Termination of tasks that used too much memory or disk
    - Still rouge tasks managed to affect the performance of other tasks on the machine
  - Solution - migrate to containers

