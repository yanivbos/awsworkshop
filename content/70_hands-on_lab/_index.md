---
title: "Hands-On Lab"
chapter: false
weight: 70
---

In this tutorial, we will build a containerized NPM application. Using the JFrog Platform, we will compile our code, build our NPM package, execute a docker build and push , security scan the image and publish to a repository. We will then deploy the image and serve the application with Amazon ECS.

![NPM Application](/images/npm-app.png)

{{% notice info %}}
<p style='text-align: left;'>
Throughout all of the lab pipelines, all artifacts are pushed to Artifactory. Artifacts in Artifactory are annotated with metadata, especially in the case of builds which come with exhaustive metadata that facilitate full traceability. This, for example, prevents the deletion of dependencies which are needed by a build. We’ll see this later on in the tutorial when we use the Tree Browser in Artifactory to view one of the artifacts produced in our build.
</p>
{{% /notice %}}




