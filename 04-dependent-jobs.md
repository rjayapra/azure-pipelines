[< Previous Challenge](./03-jobs-pipeline.md) - **[Home](../README.md)** - [Next Challenge >](./05-deployment-pipeline.md)

### Dependencies and Conditions
When you define multiple jobs in a single stage, you can specify dependencies between them. Pipelines must contain at least one job with no dependencies

Example
```

jobs:
- job: Debug
  steps:
  - script: echo hello from the Debug build
- job: Release
  dependsOn: Debug
  steps:
  - script: echo hello from the Release build
```

### Conditions
You can specify the conditions under which each job runs. By default, a job runs if it doesn't depend on any other job, or if all of the jobs that it depends on have completed and succeeded.

Example
```

jobs:
- job: A
  steps:
  - script: exit 1

- job: B
  dependsOn: A
  condition: failed()
  steps:
  - script: echo this will run when A fails

- job: C
  dependsOn:
  - A
  - B
  condition: succeeded('B')
  steps:
  - script: echo this will run when B runs and succeeds
```


#### Solution : create a dependency-job.yml with below contents and run

```
trigger:
- main
jobs:
- job: A
  steps:
  - script: "echo '##vso[task.setvariable variable=skipsubsequent;isOutput=true]false'"
    name: printvar

- job: B
  condition: and(succeeded(), ne(dependencies.A.outputs['printvar.skipsubsequent'], 'true'))
  dependsOn: A
  steps:
  - script: echo hello from B
```

