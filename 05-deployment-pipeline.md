[< Previous Challenge](./04-dependent-jobs.md) - **[Home](../README.md)** - [Next Challenge >](./06-variables-groups.md)

### Deployment job
In YAML pipelines, the deployment steps can be put in a special type of job called a *deployment job*. A deployment job is a collection of steps that are run sequentially against the environment. A deployment job and a traditional job can exist in the same stage. Azure DevOps supports the runOnce, rolling, and the canary strategies.

#### Deployment jobs provide the following benefits:

- Deployment history: You get the deployment history across pipelines, down to a specific resource and status of the deployments for auditing.
- Apply deployment strategy: You define how your application is rolled out.

#### Syntax 

```
jobs:
- deployment: string   # name of the deployment job, A-Z, a-z, 0-9, and underscore. The word "deploy" is a keyword and is unsupported as the deployment name.
  displayName: string  # friendly name to display in the UI
  pool:                # not required for virtual machine resources
    name: string       # Use only global level variables for defining a pool name. Stage/job level variables are not supported to define pool name.
    demands: string | [ string ]
  workspace:
    clean: outputs | resources | all # what to clean up before the job runs
  dependsOn: string
  condition: string
  continueOnError: boolean                # 'true' if future jobs should run even if this job fails; defaults to 'false'
  container: containerReference # container to run this job inside
  services: { string: string | container } # container resources to run as a service container
  timeoutInMinutes: nonEmptyString        # how long to run the job before automatically cancelling
  cancelTimeoutInMinutes: nonEmptyString  # how much time to give 'run always even if cancelled tasks' before killing them
  variables: # several syntaxes, see specific section
  environment: string  # target environment name and optionally a resource name to record the deployment history; format: <environment-name>.<resource-name>
  strategy:
    runOnce:    #rolling, canary are the other strategies that are supported
      deploy:
        steps: [ script | bash | pwsh | powershell | checkout | task | templateReference ]
```
 
#### Lifecycle hooks:

- `preDeploy:` Initialize resources before application deployment starts.
- `deploy:` Used to run steps that deploy your application. 
- `routeTraffic:` Used to run steps that serve the traffic to the updated version.
- `postRouteTraffic:` Used to run the steps after the traffic is routed. 
- `on: failure` or `on: success:` Used to run steps for rollback actions or clean-up.

#### Deployment Strategy
`runOnce` is the simplest deployment strategy wherein all the lifecycle hooks
`rolling deployment` replaces instances of the previous version of an application with instances of the new version of the application
`canary` an advanced deployment strategy that helps mitigate the risk involved in rolling out new versions of applications

#### Output variables
Define output variables in a deployment job's lifecycle hooks and consume them in other downstream steps and jobs within the same stage.

Access output variables across jobs using the following syntax.

For runOnce strategy: 
`$[dependencies.<job-name>.outputs['<job-name>.<step-name>.<variable-name>']]` for example, `$[dependencies.JobA.outputs['JobA.StepA.VariableA']]`
For runOnce strategy plus a resourceType: 
`$[dependencies.<job-name>.outputs['<job-name>_<resource-name>.<step-name>.<variable-name>']]`. 
For example, `$[dependencies.JobA.outputs['Deploy_VM1.StepA.VariableA']]`
For canary strategy: 
`$[dependencies.<job-name>.outputs['<lifecycle-hookname>_<increment-value>.<step-name>.<variable-name>']]`
For rolling strategy: 
`$[dependencies.<job-name>.outputs['<lifecycle-hookname>_<resource-name>.<step-name>.<variable-name>']]`


Example 

```
trigger:
    - main
    
pool:
    vmImage: ubuntu-latest
stages:
- stage: StageA
  jobs:
  - deployment: A1
    pool:
      vmImage: 'ubuntu-latest'
    environment: env1
    strategy:                  
      runOnce:
        deploy:
          steps:
          - bash: echo "##vso[task.setvariable variable=myOutputVar;isOutput=true]this is the deployment variable value"
            name: setvarStep
          - bash: echo $(System.JobName)
  
  - job: B1
    dependsOn: A1
    pool:
      vmImage: 'ubuntu-latest'
    variables:
      myVarFromDeploymentJob: $[ dependencies.A1.outputs['A1.setvarStep.myOutputVar'] ]
      
    steps:
    - script: "echo $(myVarFromDeploymentJob)"
      name: echovar
   
```