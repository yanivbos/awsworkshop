---
title: "JFrog Devops Hands-On Lab"
chapter: false
weight: 70
---

In this tutorial we’ll build a containerized microservice with an example Java and NPM two-tier web application which requires third party dependencies and Docker images used for deployment.
The Pipelines provided in this tutorial include two pre-configured pipelines that will provide you the experience in building and publishing a multi-stage web application.

1. **gradle-build** creates a Java web app which displays some images.

2. **npm-build builds** a NPM user interface that connects to the Java web app.


{{% notice info %}}
<p style='text-align: left;'>
Throughout all of the CI/CD pipelines, all artifacts are pushed to Artifactory. Artifacts in Artifactory are annotated with metadata, especially in the case of builds which come with exhaustive metadata facilitating full traceability. This, for example, prevents the deletion of dependencies which are needed by a build. We’ll see this later on in the tutorial when we use the Tree Browser in Artifactory to view one of the artifacts produced in our build.
</p>
{{% /notice %}}




