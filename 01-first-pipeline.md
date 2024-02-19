**[Home](../README.md)** - [Next Challenge >](./02-custom-pipeline.md)

### Create first pipeline in Azure DevOps

This is a step-by-step guide to use Azure Pipelines to build a sample application from a Git repository. This guide uses YAML pipelines configured with the YAML pipeline editor

### PreRequisites

1. A GitHub account where you can create a repository. Refer [GitHub]
2. An Azure DevOps organization [Azure DevOps]. 
3. An ability to run pipelines on Microsoft-hosted agents. 


### Get Started

1. Fork the repository [Pipelines Java]
1. Create the pipeline
    1. Sign in to ADO and go to your project
    1. Go to *Pipelines* -> select *New Pipelines* or *Create pipeline* if its first one
    1. Selecting GitHub as the location of your source code. Sign in when prompted to GitHub
    1. Select your repository.
    1. When redirected to GitHub to install the Azure Pipelines app. If so, select Approve & install.
    1. Azure Pipelines will analyze your repository and recommend the Maven pipeline template.
    1. Review the YAML pipeline. Select Save and run.
    1. When prompted to commit a new azure-pipelines.yml file to your repository select Save and run again.
    1. Select the build job to view the pipeline in action
    1. Now we have a working YAML pipeline (azure-pipelines.yml) in repository that's ready for customization!

When ready to make changes to pipeline, select it in the Pipelines page, and then Edit the azure-pipelines.yml file.

[GitHub]: (https://github.com/)
[Azure DevOps]: (https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops)
[Pipelines Java]: (https://github.com/MicrosoftDocs/pipelines-java)