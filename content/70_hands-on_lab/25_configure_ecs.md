---
title: "Configure ECS"
chapter: false
weight: 25
---
In the previous section, we set up JFrog Pipelines to authenticate and publish images to Artifactory. In this section, we will add the same credentials to AWS Secrets Manager and create an ECS IAM role. This will allow Amazon ECS to pull the image from Artifactory and deploy it. 

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
11. Click on your new secret _awsworkshop/jfrog-npm-app_.
12. Copy the **Secret ARN**.
![Secret ARN](/images/secret-arn.png)
12. Next we must create an IAM role that allows ECS to access these credentials. Go to [IAM Roles](https://us-east-1.console.aws.amazon.com/iam/home?#/roles).
13. Click on **Create role**.
14. Select the **Elastic Container Service** service and use case.
15. Click through the next steps and then create the role with the name _ecsWorkshop_.
16. Now click on the _ecsWorkshop_ role in the roles list.
![IAM Role](/images/iam-role.png)
17. Click on **Attach policies** and attach the **AmazonECSTaskExecutionRolePolicy** policy.
18. On the right, click on **Add inline policy**.
19. Click on the **JSON** tab and paste the following.

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

20. Substitute your **Secret ARN** from above.
21. Also substitute your \<region\> and \<aws_account_id\>. You can derive this from the **Secret ARN** format.

```
arn:aws:secretsmanager:<region>:<aws_account_id>: secret:secret_name
```

22. Click on **Review policy**.
23. Name the policy _ecsAccessToSecrets_ and create the policy.
![Inline Policy](/images/inline-policy.png)

You have now created an IAM role that will allow ECS images from Artifactory.