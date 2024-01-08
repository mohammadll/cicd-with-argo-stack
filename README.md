# cicd-with-argo-stack

## 💡 Description of the scenario
We want to combine `Argo CD`, `Argo Rollouts`, `Argo Workflows`, and `Argo Events` in the form of a scenario and project to see how they work together in the real world. First, we'll set up Nexus as our artifact repository to store Docker images and set up a private GitLab as our code repository. Argo Events help us trigger the workflow that we want to create. Then, using Argo Events Custom Resource Definitions (CRDs), we'll create a resource named "event-source" to connect to our desired project on GitLab (assuming this project includes the source code of the application). By doing this, a webhook will be automatically created on this project in GitLab. Upon any changes to this project, this webhook is called. The workflow needs to be automatically executed by Argo Events. For this reason, we'll create a resource named "sensor" whose task is to execute the workflow. But what is the task of workflows in this scenario? Using Argo Workflows, we'll write a CI pipeline whose task is to build a Docker image and push it to Nexus. Argo Workflows will then go to the Rollouts manifest and change the tag of the previous image with the new value. At this stage, when the Rollouts manifest changes, Argo CD detects the new change and applies it to the Kubernetes cluster. In this scenario, we have used the canary strategy in the Rollouts manifest

## 🔎 Follow the below steps to do this scenario

:one: **Create these namespaces in your Kubernetes cluster:**

    kubectl create ns argocd
    kubectl create ns argo-rollouts
    kubectl create ns argo-events

:two: **Install Sonatype Nexus Repository:**

    cd nexus/ && docker-compose up -d
If Nexus is up and running, create a repository named `argo-demo` with the type `docker (hosted)` and set its HTTP port to `8085`.

:three: **Add private registry to Docker/containerd:**

**Docker:**

    cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "insecure-registries": ["REPLACE_ME_WITH_NEXUS_IP_ADDRESS:8085"]
    }
    EOF
**Containerd:**

    sudo tee -a /etc/containerd/config.toml <<EOF
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."REPLACE_ME_WITH_NEXUS_IP_ADDRESS:8085"]
          endpoint = ["http://REPLACE_ME_WITH_NEXUS_IP_ADDRESS:8085"]
    EOF
Restart the service

:four: **Create a Kubernetes secret containing authentication credentials for Nexus:**

    k create secret generic -n argo-events docker-config-secret --from-file=/path/to/.docker/config.json    
