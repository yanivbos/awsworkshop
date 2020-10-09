---
title: "Execute the Initialization Pipeline"
chapter: false
weight: 50
---
The first pipeline that we will execute will initialize our environment. This pipeline will create users, groups, permissions, repositories, Xray policies and watches, Xray indexes and access federation. This prepares our JFrog Platform instance to run our gradle and npm build pipelines.

{{% notice info %}}
<p style='text-align: left;'>
This pipeline initializes the JFrog Platform for the next build pipelines by creating the necessary users, repositories, permissions and Xray configuration. It does this by using the <a href="https://www.jfrog.com/confluence/display/JFROG/REST+API" target="restapi">JFrog Platform REST APIs</a>. This is another way that you can manage and monitor the JFrog Platform. 
</p>
{{% /notice %}}

1. Go to **Pipelines** ► **My Pipelines**.
![My Pipelines](/images/MyPipelinesFinal.png)
2. Click on the **init_environment** pipeline in the **Pipelines List**.
3. Click on the first step and further click on the trigger step icon to execute this pipeline. A run will appear and it will take a few moments for JFrog Pipelines to allocate resources to execute the pipeline.
![Tigger Init Pipeline](/images/TriggerInitPipeline.png)
4. If the execution results in an error, click on the run to view the logs. Make any changes to the pipeline or integrations to correct any issues and then execute again.
![Run Error](/images/RunError.png)
5. The run will show a success status when it completes without errors.
![Run Success](/images/RunSuccess.png)

We are now ready to build our first artifacts for our application.

