[< Previous Challenge](./09-approvals-templates-pipeline.md) - **[Home](../README.md)** - [Next Challenge >](./11-templates-extends-pipeline.md)

### Templates
Templates let you define reusable content, logic, and parameters in YAML pipelines

There are two types of templates: includes and extends.

- Includes templates let you insert reusable content with a template. Content from one file is inserted into another file.
- Extends template control what is allowed in a pipeline. The template defines logic that another file must follow.

#### Limits
- No more than 100 separate YAML files may be included (directly or indirectly)
- No more than 20 levels of template nesting (templates including other templates)
- No more than 10 megabytes of memory consumed while parsing the YAML (in practice, this is typically between 600 KB - 2 MB of on-disk YAML, depending on the specific features used)

Note : Template files need to exist on your filesystem at the start of a pipeline run. You can't reference templates in an artifact. 

#### Task 1 - Solution

```
# Create File: templates/include-npm-steps.yml

steps:
- script: npm install
- script: yarn install
- script: npm run compile
```

```
# File: azure-template-pipelines.yml

jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - template: templates/include-npm-steps.yml  # Template reference
- job: Windows
  pool:
    vmImage: 'windows-latest'
  steps:
  - template: templates/include-npm-steps.yml  # Template reference
  
```

#### Task 2 - Solution

Use multi stage template file

Define stage 1 template
```
# File: templates/stages1.yml
stages:
- stage: Angular
  jobs:
  - job: angularinstall
    steps:
    - script: npm install angular
```

Define stage 2 template
```
# File: templates/stages2.yml
stages:
- stage: Build
  jobs:
  - job: build
    steps:
    - script: npm run build
```

Define job with parameters
```
# File: templates/npm-with-params.yml

parameters:
- name: name  # defaults for any parameters that aren't specified
  default: ''
- name: vmImage
  default: ''

jobs:
- job: ${{ parameters.name }}
  pool: 
    vmImage: ${{ parameters.vmImage }}
  steps:
  - script: npm install
  - script: npm test
```


```
# File: azure-pipelines-templates-stage.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Install
  jobs: 
  - job: npminstall
    steps:
    - task: Npm@1
      inputs:
        command: 'install'
  - template: templates/npm-with-params.yml  # Template reference
    parameters:
      name: Linux
      vmImage: 'ubuntu-latest'
- template: templates/stages1.yml # Template reference
- template: templates/stages2.yml # Template reference
```


#### Reference paths
Template paths can be an absolute path within the repository or relative to the file that does the including.
To use an absolute path, the template path must start with a /. All other paths are considered relative.

|
+-- fileA.yml
|
+-- dir1/
     |
     +-- fileB.yml
     |
     +-- dir2/
          |
          +-- fileC.yml

#### Reference from other repositories

Templates can be kept in other repositories. For example, there is a core pipeline that need to be used for all app pipelines, put the template in a core repo and then refer to it from each of your app repos:

```
# Repo: Contoso/BuildTemplates
# File: common.yml
parameters:
- name: 'vmImage'
  default: 'ubuntu-22.04'
  type: string

jobs:
- job: Build
  pool:
    vmImage: ${{ parameters.vmImage }}
  steps:
  - script: npm install
  - script: npm test
```

```
# Repo: Contoso/LinuxProduct
# File: azure-pipelines.yml
resources:
  repositories:
    - repository: templates
      type: github
      name: Contoso/BuildTemplates

jobs:
- template: common.yml@templates  # Template reference
```
