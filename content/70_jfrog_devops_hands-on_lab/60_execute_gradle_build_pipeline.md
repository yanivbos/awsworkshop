---
title: "Execute the Gradle Build Pipeline"
chapter: false
weight: 60
---

Our first build pipeline is a Gradle pipeline that builds our Java web app. This pipeline uses a [GradleBuild](https://www.jfrog.com/confluence/display/JFROG/GradleBuild) native Pipelines step to compile and build the code. [DockerBuild](https://www.jfrog.com/confluence/display/JFROG/DockerBuild) and [DockerPush](https://www.jfrog.com/confluence/display/JFROG/DockerPush) native steps are used to build a Docker image and push it to Artifactory. It then scans the build using the [XrayScan](https://www.jfrog.com/confluence/display/JFROG/XrayScan) native step. Then it pushes the produced artifacts to the staging repository in Artifactory along with all build information by using the [PromoteBuild](https://www.jfrog.com/confluence/display/JFROG/PromoteBuild) native step.

![Gradle Build Steps](/images/gradle-build-pipeline-steps.svg) 

1. Go to **Pipelines** ► **My Pipelines**.
![My Pipelines](/images/MyPipelinesFinal.png)
2. Click on the **gradle_build** pipeline in the **Pipelines List**.
3. Click on the **View YAML** icon to the right to view the pipeline steps discussed above.
![Gradle Build YAML](/images/gradle-build-yaml.png)
4. Click on the **Resources** tab to view the details for all the resources that are used in our pipeline. We specify the relevant input and output resources such as Git repositories, build infos and file specs. These are referenced in our pipeline.
5. Click on the **Pipeline** tab.
6. The beginning of the YAML states our pipeline name along with one read-only global environment variable.

```
name: gradle_build
configuration:
  environmentVariables:
    readOnly:
      Version: 1.1.$run_number
```
7. View the first step, _gradle\_bld\_app_. This is the GradleBuild native step that executes Gradle to build our Java webapp. The following occurs in this step:
    - We specify the Gradle command to execute, the source location, and the location for some configuration files. 
    - By using the _autoPublishBuildInfo_ parameter, we ensure the build information will be kept in Artifactory. 
    - We specify our _gitRepo\_code_ GitRepo resource as an input resource, our _Artifactory_ integration and _gradleBuildInfo_ as an output resource.
    
```
      - name: gradle_bld_app
        type: GradleBuild
        configuration:
          runtime:
            type: image
            image:
              custom:
                # THe Docker image is used to support a java 8 image for the gradle build
                name: docker.bintray.io/jfrog/pipelines-u18java
                tag: "8"
          gradleCommand:        clean artifactoryPublish -b build.gradle --stacktrace
          sourceLocation:       tutorial/step1-create-gradle-app
          configFileLocation:   .
          configFileName:       jfrog-gradle.yml
          autoPublishBuildInfo: true
          inputResources:
            - name:  gitRepo_code
          integrations:
            - name: Artifactory
          outputResources:
            - name: gradleBuildInfo
```
8. The second step, _Gradle\_docker\_build_, executes a docker build. It does the following: 
    - It will use the Docker file from the relevant location to generate the Docker image with a specific name and tag based on the value of the environment variables that were configured in the _onStart_.
    - It will use the _gitRepo\_code_ GitRepo resource as an input resource to locate the Dockerfile.
    - It will use the BuildInfo input resource from the previous step.
    - It will also use an additional input resource for downloading the Gradle-created artifact into the workspace of the running step, to be used as part of the Docker build

```
      - name: Gradle_docker_build
        type: DockerBuild
        configuration:
          affinityGroup: dock1
          dockerFileLocation: tutorial/step1-create-gradle-app
          dockerFileName: Dockerfile
          dockerImageName: ${Fullimagename}
          dockerImageTag: ${Version}
          inputSteps:
            - name: gradle_bld_app
          inputResources:
            - name: gitRepo_code
              trigger: false
            - name: gradle_fileSpec
              trigger: false
            - name: gradleBuildInfo     
              trigger: false
          integrations:
            - name: Artifactory
        execution:
          onStart:
          # export artifactory ip from internal build env var to tag the docker image
            - export domain=$(echo ${int_Artifactory_url} | awk -F[/:] '{print $4}' )
            - export Fullimagename="${domain}/docker-demo/gradle-app"


```

9. The next step will push the Docker image to the target Docker repository in Artifactory. 

```
      - name: Gradle_docker_push
        type: DockerPush
        configuration:
          affinityGroup: dock1
          targetRepository: docker-demo
          autoPublishBuildInfo: true
          integrations:
            - name: Artifactory
          inputSteps:
            - name: Gradle_docker_build
          outputResources:
            - name: docker_gradleBuild_Info
```

10. The following step scan the docker image produced in the preceding step using Xray for security vulnerabilities and license compliance. By default, a failed Xray scan will result in a failure of the step and the pipeline.

```
      - name: Gradle_docker_scan
        type: XrayScan
        configuration:
          inputResources:
            - name: docker_gradleBuild_Info
              trigger: true
          outputResources:
            - name: scanned_gradle_dockerBuild_Info 
```

11. The last step will promote the build after passing the previous Xray scan.

```
      - name: gradle_docker_promote
        type: PromoteBuild
        configuration:
          targetRepository:      docker-demo
          includeDependencies:   true
          status:                Passed
          comment:               Artifact passed Xray Scan
          copy:                  false
          inputResources:
            - name:    scanned_gradle_dockerBuild_Info
              trigger: true
          outputResources:
            - name: final_docker_gradleBuild_Info
```

12. Close the **VIEW YAML** window.
13. Click on the first step and further click on the trigger step icon to execute this pipeline. A run will appear and it will take a few moments for JFrog Pipelines to allocate resources to execute the pipeline. It will take several minutes for this pipeline to run (~10 minutes).
![Tigger Gradle Pipeline](/images/TriggerGradlePipeline.png)
14. Click on the active run to view the progress of the pipeline.
![Gradle Build Runs](/images/gradle-build-runs.png)
15. Use the pulldown to select the various steps and view the logs for each step.
![View logs](/images/view-logs.png)
16. When the run completes successfully, go to **Artifactory** ► **Builds** to view your Gradle build.
![Gradle Build](/images/gradle-build-build.png)
17. Click on **gradle_build**. This will show you a list of the last builds for your Gradle build.
![Build List](/images/builds-gradle-build.png)
18. Click on your last promoted build with a **Status** of **Passed**.
19. Click on any of the modules to view the module artifacts and dependencies.
![Module Dependencies](/images/module-dependencies.png)
20. For any artifact or dependency, click on the icon for **Show In Tree** to view it in the repository. 
![Show in Tree](/images/show-in-tree.png)
21. Go back to the **Build** view **Artifactory** ► **Builds** and explore the other tabs.
22. The **Environment** tab shows your system and environment variables.
![Environment](/images/Environment.png)
23. The **Xray Data** tab shows any security vulnerabilities and license compliance issues.
![Xray Data](/images/xray-data.png)
24. The **Diff** tab allows you to compare this build to previous builds and shows the differences.
![Diff](/images/diff.png)

This Gradle build pipeline provided an overview of a typical build, docker build and push, security scan and promotion process using Artifactory, Pipelines and Xray. You were able to execute a pipeline, monitor the progress and examine its results. Let's move on to the next pipeline to explore more features. 




