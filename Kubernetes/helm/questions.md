### 🔴 1. What is a Helm chart, and why do we use it?
Helm Chart is a package manager template for Kubernetes applications. Instead of managing multiple Kubernetes YAML files separately for Deployments, Services, Ingress, ConfigMaps, Secrets, HPA, etc., we package everything into a single reusable Helm Chart.

we rarely deploy raw manifests because maintaining hundreds of YAML files across Dev, QA, UAT, and Prod becomes difficult. With Helm, we create one chart and use different values.yaml files for each environment.

This allows us to deploy the same application with environment-specific configurations without modifying the actual manifests.

Another major advantage is versioning and rollback. Every Helm deployment is stored as a release, so if a production deployment fails, I can quickly roll back to the previous stable version using a Helm rollback command instead of redeploying everything manually.

For example, in my projects, a Java microservice may require different replica counts, resource limits, ingress hosts, and database endpoints for Dev and Prod. Using Helm, we maintain a single chart and simply pass different values files during deployment. This improves standardization, reduces manual errors, and makes deployments repeatable.
### 🔴 2. How do you convert Kubernetes manifests into a Helm project?

In real projects, when we already have Kubernetes manifests and want to use Helm, the first step is to create a Helm chart structure using helm create <chart-name> and remove the default sample templates.

Then I move the existing Deployment, Service, Ingress, ConfigMap, Secret, HPA, and other manifests into the chart's templates directory.

The next step is parameterization. Instead of hardcoding values like image tags, replica counts, resource limits, ingress hosts, ports, and environment variables, I replace them with Helm template variables and store the actual values in values.yaml.

This allows the same chart to be reused across multiple environments.

For example, if the Deployment YAML has replicas: 3, I convert it to replicas: {{ .Values.replicaCount }} and define the value inside values.yaml. Similarly, image names, namespaces, resource requests and limits, and ingress URLs are moved to values files.

After that, I validate the chart using helm lint and helm template to verify the rendered manifests. Finally, I deploy it using helm install or helm upgrade. For multiple environments, I maintain separate values files such as values-dev.yaml, values-qa.yaml, and values-prod.yaml, while keeping a single reusable chart.


### 🔴 3. What commands do you use to install, upgrade, and rollback Helm releases?

For a new deployment, I use helm install. This creates a new Helm release and deploys all Kubernetes resources defined in the chart.

                **helm install myapp ./mychart -f values-prod.yaml**

For application updates, such as a new image version, replica count change, or configuration update, I use helm upgrade. This updates only the changed resources while keeping the release history intact.

                  **helm upgrade myapp ./mychart -f values-prod.yaml**

If an issue is detected after deployment, such as application errors or performance degradation, I first check the release history and then roll back to a stable version.

                   **helm history myapp**


### 🔴 4. What does helm list do?

helm list is used to view all Helm releases deployed in a Kubernetes cluster. 

I use it to quickly check what applications are deployed, their release names, current revision numbers, namespaces, update timestamps, and deployment status."


### 🔴 5. In your Helm charts, what critical checks do you perform before an upgrade?

Before performing a Helm upgrade in production, I don't directly run helm upgrade. I perform a few critical validation checks because a failed upgrade can cause application downtime or service disruption

First, I validate the chart syntax and structure using helm lint. This helps catch template errors, missing values, and invalid chart configurations before deployment.

          helm lint ./mychart

Next, I render the manifests using helm template to verify what Kubernetes objects Helm will actually create or modify. This helps identify incorrect values, namespace issues, or unintended changes.

          helm template myapp ./mychart -f values-prod.yaml

I also compare the changes between the current release and the new release using the Helm diff plugin. this is very important because I want to know exactly what resources are changing before I deploy.

         helm diff upgrade myapp ./mychart -f values-prod.yaml

Then I verify the image tag, ConfigMap changes, Secret references, resource requests and limits, ingress configuration, and HPA settings

I also check cluster capacity to ensure the nodes have enough CPU and memory for any increase in replicas or resource limits.

During the actual deployment, I use --atomic and --wait so that if pods fail health checks or don't become ready, Helm automatically rolls back to the last stable version.

          
### 🔴 6. What is the basic folder structure of a Helm chart?

A Helm chart follows a standard directory structure that helps organize Kubernetes manifests, configurations, and metadata in a reusable way.

the most important folders I work with are templates, values.yaml, and Chart.yaml.

   **1. Chart.yaml**

    Contains chart metadata.
    Defines chart name, version, description, and application version.


       apiVersion: v2
       name: myapp
       version: 1.0.0
       appVersion: "2.5.0"
 
  **2. values.yaml**

     Default configuration file.
     Stores values such as image tag, replica count, ports, resources, ingress host, etc.
     Environment-specific files like values-dev.yaml and values-prod.yaml usually override these values.

 **3. templates/**

     Contains all Kubernetes resource templates.
     Deployment, Service, Ingress, ConfigMap, Secret, HPA, ServiceAccount, PVC, etc.
     Uses Helm templating syntax to inject values dynamically.

### 🔴 7. What is Helm chart signing, and what tools are used?

  Helm chart signing is a security feature used to verify the authenticity and integrity of a Helm chart.

  It ensures that the chart was created by a trusted source and has not been modified or tampered with after publication.

  This becomes important in enterprise environments where charts are stored in shared repositories and consumed by multiple    teams.

  When a chart is signed, Helm generates a cryptographic signature for the chart package using a GPG (GNU Privacy Guard)       private key.

  When another user or a CI/CD pipeline downloads the chart, the signature can be verified using the corresponding public      key.

  **Tools Used**
  The primary tool used is GPG (GNU Privacy Guard).
    generate key : gpg --full-generate-key


### 🔴 8. How do you share Helm charts internally?

In production environments, we don't usually share Helm charts manually through ZIP files or Git repositories. We publish them to a centralized Helm repository or artifact management platform so multiple teams and CI/CD pipelines can consume them in a standardized way.

The most common approach is storing Helm charts in enterprise artifact repositories such as JFrog Artifactory, Nexus Repository, Harbor, or OCI-compliant registries like Azure Container Registry (ACR), Amazon ECR, or GitHub Container Registry.

The workflow is typically: package the chart, push it to the internal repository, and allow teams or pipelines to pull specific versions when deploying applications."
