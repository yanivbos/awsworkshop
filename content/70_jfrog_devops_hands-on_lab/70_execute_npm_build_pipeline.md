---
title: "Execute the NPM Build Pipeline"
chapter: false
weight: 70
---

Our second build pipeline is a NPM pipeline that builds our user interface that connects to our previously built Gradle Java webapp. This pipeline uses a [NpmBuild](https://www.jfrog.com/confluence/display/JFROG/NpmBuild) native Pipelines step build the user interface components. Then it uses a generic [Bash](https://www.jfrog.com/confluence/display/JFROG/Bash) step to package the components. Next, it uses [NpmBuild](https://www.jfrog.com/confluence/display/JFROG/NpmPublish) to publish the components. Finally, similar to the Gradle build, [DockerBuild](https://www.jfrog.com/confluence/display/JFROG/DockerBuild), [DockerPush](https://www.jfrog.com/confluence/display/JFROG/DockerPush), [XrayScan](https://www.jfrog.com/confluence/display/JFROG/XrayScan) and [PromoteBuild](https://www.jfrog.com/confluence/display/JFROG/PromoteBuild) are used to promote a final docker image to the staging repository.

![Npm Build Steps](/images/npm-build-pipeline-steps.svg) 

{{% notice info %}}
<p style='text-align: left;'>
Steps are executed on build nodes. Dynamic build node pools are spun up and down on-demand by Pipelines from a cloud or Kubernetes service. This can help scale operations, and help manage costs by not incurring cloud service charges to run idle nodes. Static build node pools can also be used and are persistently available.
</p>
{{% /notice %}}

1. Go to **Pipelines** ► **My Pipelines**.
![My Pipelines](/images/MyPipelinesFinal.png)
2. Click on the **npm_build** pipeline in the **Pipelines List**.
3. Click on the **View YAML** icon to the right to view the pipeline steps discussed above.
![NPM Build YAML](/images/npm-build-yaml.png)
4. Click on the **Resources** tab to view the details for all the resources that are used in our pipeline.
5. Click on the **Pipeline** tab.
6. The first step _npm\_prep_ is an NpmBuild step. This prepares the NPM environment for building. Additionally, the following occurs:
 - The NPM repository is specified with _repositoryName_. 
 - Our Artifactory integration specifies our Artifactory server and _inputResources_ is the same source code location. 
 - _onStart_ is used to execute bash commands to set our NPM version prior to executing the step. Steps have additional [lifecycle callbacks](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Steps#PipelinesSteps-StepExecution): _onExecute_, _onSuccess_, _onFailure_ and _onComplete_.

```
      - name: npm_prep
        type: NpmBuild
        configuration:
          npmArgs:   --no-progress --no-audit
          repositoryName: npm-demo
          sourceLocation: tutorial/step2-create-ui-pkg
          integrations:
            - name: Artifactory
          inputResources:
            - name: gitRepo_code
        execution:
          onStart:
            - pushd ${res_gitRepo_code_resourcePath}/tutorial/step2-create-ui-pkg
            - npm version ${Version} --no-git-tag-version
            - popd
```

7. Next, we have a [Bash](https://www.jfrog.com/confluence/display/JFROG/Bash) step, _npm\_compile_. [Bash](https://www.jfrog.com/confluence/display/JFROG/Bash)  steps are very powerful, because they allows you to execute practically any commands. The purpose of this step is to compile our NPM package. In this step, we do the following in _onStart_:

    - [restore_run_files](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Utility+Functions#PipelinesUtilityFunctions-restore_run_files) is a [Pipeline Utility Function](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Utility+Functions) copies the current run state or context to a location to compile.
    - _npm install_ installs our packages.
    - [add_run_files](https://www.jfrog.com/confluence/display/JFROG/Pipelines+Utility+Functions#PipelinesUtilityFunctions-add_run_files) adds our temporary state or context back to the initial run state.

```
  - name: npm_compile
    type: Bash
    configuration:
      inputSteps:
        - name: npm_prep
      integrations:
        - name: Artifactory
    execution:
      onStart:
        - export tempStateLocation="$step_tmp_dir/npmSourceState"
        - restore_run_files npmBuildInputGitRepo $tempStateLocation
        - pushd $tempStateLocation/tutorial/step2-create-ui-pkg
        - npm install shelljs
        - npm install
        - add_run_files $tempStateLocation/. npmBuildInputGitRepo
        - popd
```

8. The next step, _npm\_publish_, uses [NpmPublish](https://www.jfrog.com/confluence/display/JFROG/NpmPublish) to publish a package to the Artifactory repository _npm-demo_.

```
  - name: npm_publish
    type: NpmPublish
    configuration:
      repositoryName: npm-demo
      integrations:
        - name: Artifactory
      inputSteps:
        - name: npm_compile
    execution:
      onStart:
        - export inputNpmBuildStepName="npm_prep"
```

9. The remaining steps are the same as what you saw in the Gradle build. [DockerBuild](https://www.jfrog.com/confluence/display/JFROG/DockerBuild), [DockerPush](https://www.jfrog.com/confluence/display/JFROG/DockerPush), [XrayScan](https://www.jfrog.com/confluence/display/JFROG/XrayScan) and [PromoteBuild](https://www.jfrog.com/confluence/display/JFROG/PromoteBuild) are used to build, scan, publish and promote our docker image with our NPM user interface to a staging repository.

10. Close the **VIEW YAML** window.
11. Click on the first step and further click on the trigger step icon to execute this pipeline. It will take several minutes for this pipeline to run (~10 minutes).
![Tigger Npm Pipeline](/images/TriggerNpmPipeline.png)
12. When the run is finished successfully, switch to the Builds view in Artifactory. Go to **Artifactory** ► **Builds**.
13. Click on **npm_build** in the list.
14. Then click on your most recent build.
![Npm Build Build](/images/npm-build-build.png)
15. In the **Published Modules** tab, view the set of artifacts and dependencies for your build.
![Npm Build Published Modules](/images/npm-build-published-modules.png)
16. In the **Xray Data** tab, view the security and license violations.
![Npm Build Xray Data](/images/npm-build-xray-data.png)


This NPM build pipeline provided another of example of how JFrog Pipelines, Artifactory and Xray can automate your software delivery. You explored new steps for NPM and learned how to use the flexible Bash step.
