---
title: "Configure the Artifactory Integration"
chapter: false
weight: 20
---
Similar to the GitHub Integration, in the following steps you will configure an Artifactory Integration that allows JFrog Pipelines to also access your Artifactory repositories in order to publish your artifacts and build info. You will do this by creating an API key.

1. In your JFrog Platform instance, go your profile and select **Edit Profile**.
2. Enter your password and click **Unlock** to edit the profile.
3. In the **Authentication Settings** section, click the gear icon to generate an API key.
![ApiKey](/images/ApiKey.png)
4. Copy the API key.
5. Go back to **Integrations**, select **Pipelines** ► **Integrations**.
![Pipelines](/images/Integrations.png)
6. Click on **Add an Integration**.
7. Give this integration the name _Artifactory_.
8. For the **Integration Type**, choose **Artifactory**.
9. Enter your JFrog Platform instance URL for the **url**.
10. Enter your username for the **User**.
11. Paste your Artifactory API Key into the **API Key** field.
![Artifactory Integration](/images/AddArtifactoryIntegration.png)
10. Click **Create**.

You have created an Artifactory Integration that allows JFrog Pipelines to access your Artifactory repositories. At this point, you should see the Artifactory and the GitHub Integrations in the Integrations list.

![Integrations List](/images/IntegrationsFinal.png)

