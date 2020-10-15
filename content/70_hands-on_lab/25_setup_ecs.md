---
title: "Configure AWS Secrets Manager"
chapter: false
weight: 25
---
In the previous section we set up JFrog Pipelines to authenticate and publish images to Artifactory. In this section, we will add the same credentials to AWS Secrets Manager. This will allow Amazon ECS to pull the image from Artifactory and deploy it. 

{{% notice info %}}
<p style='text-align: left;'>
Private registry authentication for ECS tasks using AWS Secrets Manager enables you to store your credentials securely and then reference them in your container definition. This allows your ECS tasks to use images from private repositories. 
</p>
{{% /notice %}}

1. Go to your [AWS Secrets Manager Console](https://console.aws.amazon.com/secretsmanager/).
2. Click on **Store a new secret**.
3. Select **Other type of secrets**.
4. Select the **Plaintext** format.
5. And paste your Artifactory username and API Key.

```
{
  "username" : "<username>",
  "password" : "<password>"
}
```

6. Click **Next**.
7. For **Secret name**, use _awsworkshop/jfrog-npm-app_.
8. Click **Next**.
9. Leave the default settings on this next **Configure automatic rotation** page and click **Next**.
10. On the **Sample code** page, click **Store**. You should now see your _awsworkshop/jfrog-npm-app_ secrets listed.

![AWS Secrets](/images/aws-secrets.png)



