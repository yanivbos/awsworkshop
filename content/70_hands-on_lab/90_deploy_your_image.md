---
title: "Deploy Your Image with ECS"
chapter: false
weight: 90
---
We are now ready to deploy your image with Amazon ECS.

{{% notice info %}}
<p style='text-align: left;'>
Private registry authentication for ECS tasks using AWS Secrets Manager enables you to store your credentials securely and then reference them in your container definition. This allows your ECS tasks to use images from private repositories. 
</p>
{{% /notice %}}

1. Go to the [Amazon ECS console first-run wizard](https://console.aws.amazon.com/ecs/home#/firstRun).
2. In the **Container definition** section, click **configure** on the **custom** option.
3. For the container name, specify _npm-app_.
4. For the **Image** specify the docker image name for your npm-app. This should be ${domain}/docker-demo/npm-app:latest. _domain_ is the JFrog Platform instance domain (_server_.jfrog.io).
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
7. Provide a **Secret name** like _awsworkshop/jfrog-npm-app_. Remember this name.
8. Click **Next**.
9. Leave the default settings on this next **Configure automatic rotation** page and click **Next**.
10. On the **Sample code** page, click **Store**. You should now see your new secret listed.
![AWS Secrets](/images/aws-secrets.png)
11. Click on your new secret.
12. Copy the **Secret ARN**.
![Secret ARN](/images/secret-arn.png)
13. Next we must create an IAM role that allows ECS to access these credentials. Go to [IAM Roles](https://us-east-1.console.aws.amazon.com/iam/home?#/roles).
14. Click on **Create role**.
15. Select the **Elastic Container Service** service and **Elastic Container Service Task** use case.
16. Click on **Create Policy**.
17. Click on the **JSON** tab and paste the following.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "<Secret ARN>",
        "arn:aws:kms:<region>:<aws_account_id>:key/key_id"     
      ]
    }
  ]
}

```

18. Substitute your **Secret ARN** from above.
19. Also substitute your \<region\> and \<aws_account_id\>. You can derive this from the **Secret ARN** format.

```
arn:aws:secretsmanager:<region>:<aws_account_id>: secret:secret_name
```

20. Click on **Review policy**.
21. Name the policy _ecsAccessToSecrets_ and create the policy.
![Inline Policy](/images/inline-policy.png)
22. Now go back to your role and search for your new policy _ecsAccessToSecrets_ and attach it. You may need to refresh the policy list. 
23. Also attach the **AmazonECSTaskExecutionRolePolicy**.
15. Click through the next steps and then create the role with the name _ecsWorkshop_.
![IAM Role](/images/iam-role.png)

You have now created an IAM role that will allow ECS images from Artifactory.