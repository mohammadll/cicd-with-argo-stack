# cicd-with-argo-stack

## 💡 Description of the scenario
We want to combine `Argo CD`, `Argo Rollouts`, `Argo Workflows`, and `Argo Events` in the form of a scenario and project to see how they work together in the real world. First, we'll set up Nexus as our artifact repository to store Docker images and set up a private GitLab as our code repository. Argo Events help us trigger the workflow that we want to create. Then, using Argo Events Custom Resource Definitions (CRDs), we'll create a resource named "event-source" to connect to our desired project on GitLab (assuming this project includes the source code of the application). By doing this, a webhook will be automatically created on this project in GitLab. Upon any changes to this project, this webhook is called. The workflow needs to be automatically executed by Argo Events. For this reason, we'll create a resource named "sensor" whose task is to execute the workflow. But what is the task of workflows in this scenario? Using Argo Workflows, we'll write a CI pipeline whose task is to build a Docker image and push it to Nexus. Argo Workflows will then go to the Rollouts manifest and change the tag of the previous image with the new value. At this stage, when the Rollouts manifest changes, Argo CD detects the new change and applies it to the Kubernetes cluster. In this scenario, we have used the canary strategy in the Rollouts manifest
