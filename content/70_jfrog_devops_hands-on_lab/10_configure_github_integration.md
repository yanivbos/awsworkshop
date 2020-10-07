---
title: "Configure the GitHub Integration"
chapter: false
weight: 10
---

In order for JFrog Pipelines, to get access to the code in your new repositories, we must first set up a Pipelines GitHub integration. This allows Pipelines to authenticate and get access to your GitHub repositories. To do this, we create a GitHub Personal Access Token with the correct permissions.

1. Go to your [GitHub Personal Access Tokens settings page](https://github.com/settings/tokens).
2. Click on **Generate new token**.
3. Provide a name for the token.
4. Configure the token for the following scopes.

```
* repo (all)
* admin:repo_hook (read, write)
* admin:public_key (read, write)
```
5. Click **Generate token**.
6. Copy the token.
7. Go to your JFrog Platform instance at _https://[server name].jfrog.io_. Refer to your _JFrog Free Subscription Activation_ email if needed.
8. Login to your JFrog Platform instance with your credentials.
![Login](/images/Login.png)
9. Once logged into the environment, you will be presented with the landing page.
![Landing](/images/Landing.png)
10. On the left sidebar menu, select **Pipelines** ► **Integrations**.
![Pipelines](/images/Integrations.png)
10. Click on **Add an Integration**.
11. Give this integration the name _GitHub_.
12. For the **Integration Type**, choose **GitHub**.
13. Paste your GitHub Personal Access Token into the **Token** field.
![GitHub Integration](/images/AddGitHubIntegration.png)
14. Click **Create**.

You have created a GitHub Integration that allows JFrog Pipelines to access your GitHub repositories.




