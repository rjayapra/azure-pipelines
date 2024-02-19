[< Previous Challenge](./06-variables-groups.md) - **[Home](../README.md)** - [Next Challenge >](./08-keyvault-secret-pipeline)

### Runtime Parameters
Runtime parameters let you have more control over what values can be passed to a pipeline. With runtime parameters you can:

- Supply different values to scripts and tasks at runtime
- Control parameter types, allowed ranges, and defaults
- Dynamically select jobs and stages with template expressions

### Task 1 : Use Parameters in pipeline
#### Solution
```
parameters:
- name: image
  displayName: Pool Image
  type: string
  default: ubuntu-latest
  values:
  - windows-latest
  - ubuntu-latest
  - macOS-latest

trigger: none

jobs:
- job: build
  displayName: build
  pool: 
    vmImage: ${{ parameters.image }}
  steps:
  - script: echo building $(Build.BuildNumber) with ${{ parameters.image }}
```