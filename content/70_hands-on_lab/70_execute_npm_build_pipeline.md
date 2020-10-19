---
title: "Execute the NPM Build Pipeline"
chapter: false
weight: 70
---

The **npm_build** pipeline builds our web application. This pipeline uses a [NpmBuild](https://www.jfrog.com/confluence/display/JFROG/NpmBuild) native Pipelines step build the user interface components. Next, it uses [NpmPublish](https://www.jfrog.com/confluence/display/JFROG/NpmPublish) to publish the components. [DockerBuild](https://www.jfrog.com/confluence/display/JFROG/DockerBuild) and [DockerPush](https://www.jfrog.com/confluence/display/JFROG/DockerPush) native steps are used to build a Docker image and push it to Artifactory. It then scans the build using the [XrayScan](https://www.jfrog.com/confluence/display/JFROG/XrayScan) native step. Then it pushes the produced artifacts to the "staging" repository in Artifactory along with all build information by using the [PromoteBuild](https://www.jfrog.com/confluence/display/JFROG/PromoteBuild) native step.

![Npm Build Steps](/images/npm-build-pipeline-steps.svg) 

{{% notice info %}}
<p style='text-align: left;'>
A Step is a unit of execution in a pipeline. It is triggered by some event and uses resources to perform an action as part of the pipeline. Steps take Inputs in the form of Integrations or Resources, execute tasks that perform the operations necessary and then produce Outputs. These Outputs can become Inputs to other steps and so on forming a dependency-based, event-driven pipeline.
<br/>
<br/>
Steps are executed on build nodes. Dynamic build node pools are spun up and down on-demand by Pipelines from a cloud or Kubernetes service. This can help scale operations, and help manage costs by not incurring cloud service charges to run idle nodes. Static build node pools can also be used and are persistently available.
</p>
{{% /notice %}}

1. Go to **Pipelines** ► **My Pipelines**.
![My Pipelines](/images/pipelines-list.png)
2. Click on the **npm_build** pipeline in the **Pipelines List**.
3. Click on the **View YAML** icon to the right to view the pipeline steps discussed above.
![NPM Build YAML](/images/npm-build-yaml.png)
4. Click on the **Resources** tab to view the details for all the resources that are used in our pipeline. We specify the relevant input and output resources such as Git repositories, build infos and file specs. These are referenced in our pipeline.
5. Click on the **Pipeline** tab.
6. The first step _npm\_prep_ is an NpmBuild step. This prepares the NPM environment for building. Additionally, the following occurs:
 - The NPM repository is specified with _repositoryName_. 
 - Our Artifactory integration specifies our Artifactory server and _inputResources_ is the same source code location. 

```
      - name: npm_prep
        type: NpmBuild
        configuration:
          npmArgs:   --no-progress --no-audit
          repositoryName: npm-demo
          sourceLocation: workshop-app
          integrations:
            - name: Artifactory
          inputResources:
            - name: gitRepo_code
```

7. The next step, _npm\_publish_, uses [NpmPublish](https://www.jfrog.com/confluence/display/JFROG/NpmPublish) to publish a package to the Artifactory repository _npm-demo_.

```
    - name: npm_publish
    type: NpmPublish
    configuration:
      repositoryName: npm-demo
      integrations:
        - name: Artifactory
      inputSteps:
        - name: npm_compile
```

8. The _npm\_docker\_build_ step executes a docker build. It does the following: 
    - _onStart_ is used to execute bash commands prior to executing the step. Steps have additional [lifecycle callbacks](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Steps#PipelinesSteps-StepExecution): _onExecute_, _onSuccess_, _onFailure_ and _onComplete_.
    - It will use the Docker file from the relevant location to generate the Docker image with a specific name and tag based on the value of the environment variables that were configured in the _onStart_.
    - It will use the _gitRepo\_code_ GitRepo resource as an input resource to locate the Dockerfile.
    - It will use the BuildInfo input resource from the previous step.
    - It will create a docker image ${domain}/docker-demo/npm-app. _domain_ is the JFrog Platform instance domain (\<server\>.jfrog.io).

```
      - name: npm_docker_build
        type: DockerBuild
        configuration:
          affinityGroup: bldGroup
          dockerFileLocation: workshop-app
          dockerFileName: Dockerfile
          dockerImageName: ${Fullimagename}  
          dockerImageTag: ${Version}
          inputSteps:
            - name: npm_publish
          inputResources: 
            - name: gitRepo_code
          integrations:
            - name: Artifactory
        execution:
          onStart:
            - pushd ${res_gitRepo_code_resourcePath}/workshop-app
            # Creating a Folder for the fileSpec Target
            - mkdir -p npm_results
            - popd
            - export domain=$(echo ${int_Artifactory_url} | awk -F[/:] '{print $4}' )
            - export Fullimagename="${domain}/docker-demo/npm-app"
          onSuccess:
            - echo "Congrats The Docker image was build"
```


9. The next step will push the Docker image to the target Docker repository in Artifactory. 

```
      - name: Npm_docker_push
        type: DockerPush
        configuration:
          affinityGroup: bldGroup
          targetRepository: docker-demo
          autoPublishBuildInfo: true
          integrations:
            - name:  Artifactory
          inputSteps:
            - name: npm_docker_build
          outputResources:
            - name: docker_npmBuild_Info
```

10. The following step uses Xray to scan the docker image for security vulnerabilities and license compliance. By default, a failed Xray scan will result in a failure of the step and the pipeline.

```
      - name: npm_docker_scan
        type: XrayScan
        configuration:
          inputResources:
            - name: docker_npmBuild_Info
              trigger: true
          outputResources: 
            - name: scanned_npm_dockerBuild_Info
```

11. The last step will promote the build to the "staging" repository after passing the previous Xray scan.

```
      - name: npm_docker_promote
        type: PromoteBuild
        configuration:
          targetRepository:      docker-demo-prod-local
          includeDependencies:   true   
          status:                Passed
          comment:               Artifact passed Xray Scan
          copy:                  false
          inputResources:
            - name:    scanned_npm_dockerBuild_Info
              trigger: true       
          outputResources:
            - name: final_docker_npmBuild_Info 
```

12. Close the **VIEW YAML** window.
13. Click on the first step and further click on the trigger step icon to execute this pipeline. It will take several minutes for this pipeline to run (~15-20 minutes).
![Trigger Npm Pipeline](/images/trigger-npm-pipeline.png)
14. When the run finishes successfully, switch to the **Packages** view in Artifactory. Go to **Artifactory** ► **Packages**.
15. Type _npm-app_ and search for the docker image that you just built.
16. Then click on your docker _npm-app_ listing.
![Npm App Package](/images/npm-app-package.png)
17. This will show a list of the docker images for this build. Click on the _latest_ version that you just built.
![Npm Build Published Modules](/images/npm-app-versions.png)
18. In the **Xray Data** tab, view the security violations.
![Npm Build Xray Data](/images/npm-build-xray-data.png)
19. Click on any violation to see the details and impact in the **Issue Details** tab.
![Npm Build Xray Data](/images/npm-build-xray-detail.png)
20. Close the **Issue Details** tab.
21. View the Docker configuration for the image in the **Docker Layers** tab.
22. On the **Builds** tab, click on _npm\_build_ in the list.
![Npm Build List](/images/npm-build-list.png)
23. Then click on your most recent build.
24. In the **Published Modules** tab, view the set of artifacts and dependencies for your build.
![Npm Published Modules](/images/npm-published-modules.png)


The **npm_build** pipeline provided an overview of a typical build, docker build and push, security scan and promotion process using Artifactory, Pipelines and Xray. You were able to execute a pipeline, monitor the progress and examine its results. You explored new steps for NPM.

Next, we will deploy your docker image from the "staging" repository using ECS.
