Argo CD ApplicationSet with Dynamic List Generator
This README provides an explanation and example of how to use a dynamic list generator with Go templating in Argo CD's ApplicationSet. This example demonstrates the generation of a list of application parameters, including env and name for each application, from a YAML configuration.

Overview
Argo CD allows the management of Kubernetes resources using a GitOps approach. With the use of ApplicationSet, you can automate the creation of multiple Argo CD Applications based on dynamic parameters. In this example, we use Go templating to dynamically generate a list of applications and their parameters (environment and name) from a specified YAML file.

Key Concepts
ApplicationSet: A controller in Argo CD that allows you to manage multiple applications with a single definition.
Matrix Generator: A generator type in ApplicationSet that allows combining multiple generators.
List Generator: Another generator type that creates a list of parameters to create new applications.

Example
YAML Configuration for ApplicationSet
The following example demonstrates how to use the list generator in an ApplicationSet to dynamically generate application parameters (env and name).

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-helm-applicationset-git-generators-scbconnect
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  generators:
    - git:
        repoURL: https://github.com/nopponcha/argocd-demo.git  # Git repository URL
        revision: HEAD  # Git revision (branch, commit, or tag)
        directories:
          - path: helm-value/scbconnect/*/*  # Directory pattern in the repository to scan for applications
  template:
    metadata:
      name: "{{path[2]}}-{{path.basename}}"  # Application name based on path in Git
    spec:
      project: default  # Argo CD project
      sources:
        - repoURL: https://github.com/nopponcha/helm-chart.git  # Helm chart repository URL
          targetRevision: HEAD  # Helm chart revision (branch, commit, or tag)
          path: charts  # Path to the Helm chart in the repository
          helm:
            valueFiles:
              - "$values/helm-value/scbconnect/{{path[2]}}/{{path.basename}}/values.yaml"  # Dynamic values.yaml file
        - repoURL: https://github.com/nopponcha/argocd-demo.git  # Values repository URL
          targetRevision: HEAD  # Git revision for values
          ref: values  # Reference for values
      destination:
        server: https://kubernetes.default.svc  # Kubernetes cluster URL
        namespace: "{{path.basename}}"  # Kubernetes namespace dynamically created from the Git path
      syncPolicy:
        automated:
          selfHeal: true  # If replicas are manually scaled, Argo CD will revert to the desired state defined in the Git repository
          prune: true  # If resources are deleted from the Git repository, they will be removed from the cluster
        syncOptions:
          - CreateNamespace=true  # Automatically create namespaces if they don't exist
          - DisableHelmChartCache=true  # Disable Helm chart caching to always fetch the latest chart version

Explanation of Key Sections
1. Git Generator Configuration:

```yaml
generators:
  - git:
      repoURL: https://github.com/nopponcha/argocd-demo.git
      revision: HEAD
      directories:
        - path: helm-value/scbconnect/*/*

- The repoURL specifies the Git repository where Argo CD will pull the configuration from.
- The revision is set to HEAD, meaning the latest commit from the default branch will be used.
- The directories field specifies a path pattern (helm-value/scbconnect/*/*) to search for directories that match the pattern. Each directory matching this pattern will generate a new application.

2. Application Template:

```yaml
template:
  metadata:
    name: "{{path[2]}}-{{path.basename}}"

- The metadata.name dynamically generates the application name based on the Git path. Here, path[2] refers to the third element in the path, and path.basename refers to the base name of the directory.

3. Source Configuration for Helm:

```yaml
sources:
  - repoURL: https://github.com/nopponcha/helm-chart.git
    targetRevision: HEAD
    path: charts
    helm:
      valueFiles:
        - "$values/helm-value/scbconnect/{{path[2]}}/{{path.basename}}/values.yaml"

- The repoURL specifies the Git repository that contains the Helm charts.
- The targetRevision is set to HEAD, meaning it will use the latest commit.
- The valueFiles path uses the dynamic Git path to specify the values file for the Helm chart.

4. Destination Configuration:

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: "{{path.basename}}"

- The namespace dynamically uses the base name of the directory from the Git path to set the Kubernetes namespace for the application.

5. Sync Policy:
- selfHeal: true ensures that Argo CD will automatically restore the desired state if any manual changes are made to the application replicas.
- prune: true will remove resources from the cluster if they are deleted from the Git repository.
syncOptions specifies additional options for syncing, including creating missing namespaces and disabling Helm chart caching.

Final Output
This configuration will result in the creation of multiple Argo CD applications based on the directories found in the specified Git repository (helm-value/scbconnect/*/*). For each directory, an application will be created with the following details:

The name of the application will be dynamically generated from the Git path.
The Helm chart and values.yaml will be used to deploy the application.

Conclusion
This example demonstrates how to use the Git Generator in Argo CD's ApplicationSet to automatically create multiple applications based on directories in a Git repository. By using dynamic path variables, we can customize the deployment of Helm charts and values to Kubernetes, making the process of managing multiple environments more efficient and automated.