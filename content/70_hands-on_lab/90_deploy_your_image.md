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
2. In the **Container definition** section, click **Configure** on the **custom** option.
3. For the container name, specify _npm-app_.
4. For the **Image** specify the docker image name for your npm-app. This should be ${domain}/docker-demo/npm-app:latest. _domain_ is the JFrog Platform instance domain (_server_.jfrog.io).
5. Check **Private repository autentication**.
6. For **Secrets Manager ARN** paste the **Secrets ARN** from the [ECS permissions step](./80_configure_ecs_permissions.md).
7. For port mapping, specify 443.
8. Click **Update**.
![ECS Container](/images/ecs-container.png)
9. Click **Edit** on the **Task definition**.
10. for the **Task definition name**, specify _deploy-npm-app_.
11. For the **Task execution role** specify the ECS role that you created, _ecsWorkshop_.
![ECS Task Definition](/images/ecs-task-definition.png)
12. Click **Save**.
13. Click **Next**.
14. For **Define your service**, ensure **Application Load Balancer** is selected and port 444 is listed.
![ECS Service](/images/ecs-service.png)
15. Click **Next**.
16. For **Configure your cluster**, specify _npm-app-cluster_ for your **Cluster name**.
17. Click **Next**.
18. Review your configuration.
19. Click **Create** after you validated your configuration.
20. Wait for your AWS services to be deployed.
21. When ready, click on your deployed service.
![NPM APP Service](/images/npm-app-service.png)
22. Click on the **Tasks** tab.
![NPM APP Service Tasks](/images/npm-app-service-tasks.png)
23. Click on the _deploy-npm-app_ task.
24. On the **Details** page of the task, locate the **Public IP**.
![NPM APP Task Detail](/images/npm-app-service-task-detail.png)
25. Click on the **ENI Id** network interface link.

25. In your browser, go to https://<Public IP> to view your deployed web application. Click through the self-signed certificate warning.




You have now created an IAM role that will allow ECS images from Artifactory.