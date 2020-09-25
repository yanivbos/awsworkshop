---
title: "Artifactory"
chapter: false
weight: 10
---

1. Now we are going to populate the JFrog Platform with the tutorial.
Please fork the following [JFrog Tutorial GitHub project] (https://github.com/shimib/Horae).

2. Login into Artifactory.

3. Go into Pipelines -> Integrations.

4. Click "Add an Integration".

5. Select "Artifactory" as the integration type and name it: *Artifactory*.

6. Provide your Artifactory URL and your Credentials.

7. Add another Pipelines integration:

8. Integration type: GitHub, name: *GitHub*.

9. Provide your GitHub personal access token (follow [these instructions](https://www.jfrog.com/confluence/display/JFROG/GitHub+Integration) for the creation of the token).

10. Go to Pipelines -> Pipeline Sources.

11. Click *Add Pipeline Source*.

12. Select *Single Branch*.

13. Select the integration named: *GitHub*.

14. Enter the repository you've forked the project into (in the pre-requisites step).

15. In "Pipeline Config File Filter" enter __pipelines/base_.*yml__

16. Go to Pipelines -> My Pipelines.

17. Select the *init_environment* pipeline.

18. Click on the "Create_scaffholding" step.

19. Click the "trigger this step" icon.

{{% notice note %}}
Wait for the Run to finish before moving to the next chapter.
When this run finishes, your tutorial environment is ready to be used.
{{% /notice %}}

