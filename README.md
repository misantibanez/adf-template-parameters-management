# Intro
Azure Data Factory template parameters management is a guideline of how to manage parameters in Azure Data Factory since it could affect CI/CD workflow.

# Current scenario

The client has a pipeline that is executed with the configurations sent by a trigger. In that sense, it has dozens of triggers that are executed according to the file that reaches a storage account.

Likewise, they have 2 environments: development and production. They handle deployment automatically between environments.

However, by continuing to build triggers, they reached the limit of parameters (256) supported by the ARM template. This is because several parameters are created per trigger, multiplying quickly.

<img src="https://user-images.githubusercontent.com/92458075/188023623-d85ee468-984f-481f-8ed2-c829ec534457.png" alt="drawing" width="500"/>
![image](https://user-images.githubusercontent.com/92458075/188023623-d85ee468-984f-481f-8ed2-c829ec534457.png)


# Alternative 1 (workaround)

A quick way out is to remove the parameters corresponding to the triggers from the ARM template. However, this will involve adding a step that replaces the Storage Account in the production environment with the corresponding one in production. Later, they might even hit the storage event type limit of 500, which will require revisiting the scenario.

script provided in source folder

<img src="https://user-images.githubusercontent.com/92458075/188023812-3f6d72a7-9e8f-4c5a-981c-614cd248a467.png" alt="drawing" width="500"/>
![image](https://user-images.githubusercontent.com/92458075/188023812-3f6d72a7-9e8f-4c5a-981c-614cd248a467.png)

# Alternative 2 (recommended)

Based on best practices, when the ARM template parameter limit is reached, you are suggested to review and remove those you don't need. In this case, we do need the parameters in the template because this allows us to automatically change the storage account in the release pipeline.
Additionally, the trigger generates its own metadata, storing in it the name of the storage account, container, file, among others. This metadata can be used in the pipeline as logic prior to the execution of the main activities.
In this sense, the idea is to use an event trigger pipeline and obtain the metadata in the main pipeline to execute the activities according to the logic in the triggers. Seeking to transfer the logic of the triggers to the level of metadata and execution of the pipeline.

script provided in source folder (same as current scenario)

<img src="https://user-images.githubusercontent.com/92458075/188023906-321ba0ea-e680-496b-900a-031f159b7934.png" alt="drawing" width="500"/>
![image](https://user-images.githubusercontent.com/92458075/188023906-321ba0ea-e680-496b-900a-031f159b7934.png)

# References

### Parameter limitation
Your factory is so large that the default Resource Manager template is invalid because it has more than the maximum allowed parameters (256).
To handle custom parameter 256 limit, there are three options:
Use the custom parameter file and remove properties that don't need parameterization, i.e., properties that can keep a default value and hence decrease the parameter count.
Refactor logic in the dataflow to reduce parameters, for example, pipeline parameters all have the same value, you can just use global parameters instead.
Split one data factory into multiple data flows.

Link: [Custom parameters in a Resource Manager template - Azure Data Factory | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-resource-manager-custom-parameters)

### Storage Event Triggers limitations

The Storage Event Trigger currently supports only Azure Data Lake Storage Gen2 and General-purpose version 2 storage accounts. Due to an Azure Event Grid limitation, Azure Data Factory only supports a maximum of 500 storage event triggers per storage account. If you hit the limit, please contact support for recommendations and increasing the limit upon evaluation by Event Grid team.

Link: [Create event-based triggers - Azure Data Factory & Azure Synapse | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-factory/how-to-create-event-trigger?tabs=data-factory)

### Incremental Deployment (DevOps release pipeline)

In Complete deployment mode, resources that exist in the resource group but aren't specified in the new Resource Manager template will be deleted. For more information, please refer to Azure Resource Manager Deployment Modes.

Link: [Automate continuous integration - Azure Data Factory | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-automate-azure-pipelines#updating-active-triggers)

### DataFactory Publish

By design, Data Factory doesn't allow cherry-picking of commits or selective publishing of resources. Publishes will include all changes made in the data factory.
Data factory entities depend on each other. For example, triggers depend on pipelines, and pipelines depend on datasets and other pipelines. Selective publishing of a subset of resources could lead to unexpected behaviors and errors.
On rare occasions when you need selective publishing, consider using a hotfix. For more information, see Hotfix production environment.

Link: [Continuous integration and delivery - Azure Data Factory | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery)

### Links
- https://stackoverflow.com/questions/62052791/how-to-get-the-name-of-the-file-that-triggered-the-azure-data-factory-pipeline 
- https://docs.microsoft.com/en-us/azure/data-factory/concepts-pipeline-execution-triggers#trigger-execution-with-json 
- https://docs.microsoft.com/en-us/powershell/module/az.datafactory/set-azdatafactoryv2trigger?view=azps-7.2. 0
- https://docs.microsoft.com/en-us/azure/data-factory/control-flow-expression-language-functions#expressions 
- https://docs.microsoft.com/en-us/azure/data-factory/how-to-use-trigger-parameterization 
- https://docs.microsoft.com/en-us/azure/data-factory/author-global-parameters 
- https://stackoverflow.com/questions/68839698/global-parameters-for-trigger-body-properties-and-using-split-function-on-trigg 
- http://www.mutazag.com/blog/code/tutorial/ADF-custom-paramaters/ 
- https://docs.microsoft.com/en-us/azure/templates/microsoft.datafactory/2018-06-01/factories/triggers?tabs=bicep 
- https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview
- https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-sample-script 
