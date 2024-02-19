[< Previous Challenge](./02-custom-pipeline.md) - **[Home](../README.md)** - [Next Challenge >](./04-dependent-jobs.md)

### Pipeline Jobs

A job is a series of steps that run sequentially as a unit. In other words, a job is the smallest unit of work that can be scheduled to run.

When a pipeline has a single job you don't have to explicitly use the job keyword unless you're using a template.

Example
```
pool:
  vmImage: 'ubuntu-latest'
steps:
- bash: echo "Hello world"
```


Multiple jobs

```
jobs:
- job: A
  steps:
  - bash: echo "A"

- job: B
  steps:
  - bash: echo "B"
```
When pipeline has multiple stageswith multiple jobs ;

```
stages:
- stage: A
  jobs:
  - job: A1
  - job: A2

- stage: B
  jobs:
  - job: B1
  - job: B2
```

#### Syntax

```
- job: string  # name of the job, A-Z, a-z, 0-9, and underscore
  displayName: string  # friendly name to display in the UI
  dependsOn: string | [ string ]
  condition: string
  strategy:
    parallel: # parallel strategy
    matrix: # matrix strategy
    maxParallel: number # maximum number simultaneous matrix legs to run
    # note: `parallel` and `matrix` are mutually exclusive
    # you may specify one or the other; including both is an error
    # `maxParallel` is only valid with `matrix`
  continueOnError: boolean  # 'true' if future jobs should run even if this job fails; defaults to 'false'
  pool: pool # agent pool
  workspace:
    clean: outputs | resources | all # what to clean up before the job runs
  container: containerReference # container to run this job inside
  timeoutInMinutes: number # how long to run the job before automatically cancelling
  cancelTimeoutInMinutes: number # how much time to give 'run always even if cancelled tasks' before killing them
  variables: { string: string } | [ variable | variableReference ] 
  steps: [ script | bash | pwsh | powershell | checkout | task | templateReference ]
  services: { string: string | container } # container resources to run as a service container
  uses: # Any resources (repos or pools) required by this job that are not already referenced
    repositories: [ string ] # Repository references to Azure Git repositories
    pools: [ string ] # Pool names, typically when using a matrix strategy for the job
```

#### Types of jobs

- Agent pool jobs : run on an agent in an agent pool.
    - When using MS hosted agent , each job in pipeline gets a fresh agent
    - Use *demands* when using self-hosted agent and specify agent capabilities
    ```
    pool:
      name: myPrivateAgents
      demands:
      - agent.os -equals Darwin
      - anotherCapability -equals somethingElse
    steps:
    - script: echo hello world
    ```
- Server jobs run on the Azure DevOps Server. A server job doesn't require an agent or any target computers. Supported tasks include
    - Delay task
    - Invoke Azure Function task
    - Invoke REST API task
    - Manual Validation task
    - Publish To Azure Service Bus task
    - Query Azure Monitor Alerts task
    - Query Work Items task
    
        ```
        jobs:
        - job: string
          timeoutInMinutes: number
          cancelTimeoutInMinutes: number
          strategy:
            maxParallel: number
            matrix: { string: { string: string } }
        
          pool: server # note: the value 'server' is a reserved keyword which indicates this is an agentless job
        ```
- Container jobs run in a container on an agent in an agent pool. On Linux and Windows agents, jobs may be run on the host or in a container.
    - Linux based containers
        ```
        pool:
          vmImage: 'ubuntu-latest'
        
        container: ubuntu:18.04
        
        steps:
        - script: printenv
        ```
        - Windows containers
        ```
        pool:
          vmImage: 'windows-2019'
        
        container: mcr.microsoft.com/windows/servercore:ltsc2019
        
        steps:
        - script: set
        ```   

#### Solution
Run the sample container jobs pipeline. Create a pipeline file container-jobs-pipeline.yml
```
trigger:
- main

pool:
  vmImage: ubuntu-latest

resources:
  containers:
  - container: u16
    image: ubuntu:16.04

  - container: u18
    image: ubuntu:18.04

  - container: u20
    image: ubuntu:20.04

jobs:
- job: Delay
  pool: server
  steps:
  - task: Delay@1
    inputs:
      delayForMinutes: '2'

- job: RunInContainer
  pool:
    vmImage: 'ubuntu-latest'

  strategy:
    matrix:
      ubuntu16:
        containerResource: u16
      ubuntu18:
        containerResource: u18
      ubuntu20:
        containerResource: u20

  container: $[ variables['containerResource'] ]

  steps:  
    - script: printenv

```



