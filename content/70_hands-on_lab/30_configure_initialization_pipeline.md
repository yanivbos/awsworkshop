---
title: "Configure the Initialization Pipeline"
chapter: false
weight: 30
---

Next, we will update the lab pipelines to add your new GitHub and Artifactory integrations. In previous steps, you [forked and cloned the lab repository.]() We will modify the initialization pipeline in your forked Hoare repository to add these integrations.

1. In your local git directory, open awsworkshop/jfrog_pipelines/init-jfrog.yml in an editor.
2. Update the resources section of the file to use your new forked repository. Change the _path_ to use your username.

```
resources:  
  - name: gitRepo_code  
    type: GitRepo  
    configuration:  
      path: [your_Github_username]/awsworkshop  <<<--- CHANGE HERE
      gitProvider: GitHub  
```

3. Save your changes.

4. In your terminal, git add, commit and push your changes.

```
$ git add .
$ git commit -m 'Updated repository path.'
$ git push
```
5. Go to https://github.com/[username]/awsworkshop/blob/master/jfrog_pipelines/init-jfrog.yml and verify that your changes were made and pushed to your GitHub repository.

We are now ready to add and execute your pipelines with JFrog Pipelines.