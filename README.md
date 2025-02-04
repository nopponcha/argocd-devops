# Argo CD ApplicationSet with Dynamic List Generator

## Overview

This README provides an explanation and example of how to use a dynamic list generator with Go templating in Argo CD's **ApplicationSet**. This example demonstrates the generation of a list of application parameters (including `env` and `name` for each application) from a specified YAML configuration.

Argo CD uses a GitOps approach for managing Kubernetes resources. The **ApplicationSet** feature allows you to automate the creation of multiple Argo CD Applications based on dynamic parameters, making it easier to manage and scale deployments.

### Key Concepts

- **ApplicationSet**: A controller in Argo CD that enables you to manage multiple applications with a single definition.
- **Matrix Generator**: A generator type in ApplicationSet for combining multiple generators.
- **List Generator**: A generator that creates a list of parameters used to generate new applications.

---

## Example: Using a Git Generator for ApplicationSet

The following example demonstrates how to use the **Git Generator** with the **ApplicationSet** to dynamically generate application parameters like `env` and `name` based on the contents of a Git repository.

### YAML Configuration for ApplicationSet

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
```
## Explanation of Key Sections
1. Git Generator Configuration
- **repoURL**: Specifies the Git repository where Argo CD will pull the configuration from.
- **revision**: Set to HEAD, meaning the latest commit from the default branch will be used.
- **directories**: Defines a path pattern (helm-value/scbconnect/*/*) to search for directories in the repository that match the pattern. Each matching directory will generate a new application.

2. ## Application Template
**metadata.name**: Dynamically generates the application name from the Git path. path[2] refers to the third element in the path, and path.basename refers to the base name of the directory.

3. ## Source Configuration for Helm
**repoURL**: Specifies the Git repository containing the Helm charts.
valueFiles: Uses a dynamic Git path to specify the location of the values.yaml file for the Helm chart.

4. ## Destination Configuration
**namespace**: Dynamically sets the Kubernetes namespace based on the base name of the directory in the Git path.

5. ## Sync Policy
**selfHeal: Ensures that Argo CD will automatically restore the desired state if any manual changes are made to the application replicas.**
**prune: Removes resources from the cluster if they are deleted from the Git repository.**
**syncOptions: Additional options for syncing, such as creating missing namespaces and disabling Helm chart caching.**

## Final Output
This configuration will result in the creation of multiple Argo CD applications based on the directories found in the specified Git repository (helm-value/scbconnect/*/*). For each directory, an application will be created with the following details:

The application name will be dynamically generated from the Git path.
The Helm chart and corresponding values.yaml will be used to deploy the application.

### Conclusion
This example demonstrates how to use the Git Generator in Argo CD's ApplicationSet to automatically create multiple applications based on directories in a Git repository. By using dynamic path variables, we can customize the deployment of Helm charts and values to Kubernetes, making the process of managing multiple environments more efficient and automated.

## Example Using a YAML Configuration for ApplicationSet
The following example demonstrates how to use the list generator in an ApplicationSet to dynamically generate application parameters (env and name).

### YAML Configuration for ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-helm-applicationset-yaml-list-generators
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
    - list:
        elementsYaml: |
          {{- $appsList := list }}  # Initialize an empty list
          {{- range .scb.scbconnect.applications }}  # Iterate over applications
            {{- $env := .env }}  # Store the environment value
            {{- range .apps }}  # Iterate over the apps
              {{- $appsList = append $appsList (dict "env" $env "name" .name) }}  # Append env and app name to the list
            {{- end }}
          {{- end }}
          {{- toJson $appsList }}  # Convert the list to JSON format
  template:
    metadata:
      name: '{{ .name }}-{{ .env }}'
    spec:
      project: default
      sources:
        - repoURL: '{{ .scb.scbconnect.common.helmRepoUrl }}'
          targetRevision: '{{ .scb.scbconnect.common.helmTargetRevision }}'
          path: '{{ .scb.scbconnect.common.chart }}'
          helm:
            valueFiles:
              - '$values/helm-value/scbconnect/{{ .name }}/{{ .env }}/values.yaml'
        - repoURL: '{{ .scb.scbconnect.common.helmValueRepoUrl }}'
          targetRevision: main
          ref: values
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{ .env }}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - DisableHelmChartCache=true
```
## Explanation of Key Sections

1. **Initialization of $appsList**:
```yaml
{{- $appsList := list }}
```
- **This initializes an empty list called $appsList which will be used to store application parameters.**

2. **Iterating over applications**:
```yaml
{{- range .scb.scbconnect.applications }}
```
- **The range function iterates over the applications defined in your configuration (e.g., dev, sit).**

3. **Storing environment value**:
```yaml
{{- $env := .env }}
```
- **Stores the environment value (env) of the current application (e.g., "dev" or "sit").**

4. **Iterating over each app in the current environment**:
```yaml
{{- range .apps }}
```
- **The range function iterates over the apps in the current environment, such as app-be and app-fe.**

5. **Appending app parameters to the list**:
```yaml
{{- $appsList = append $appsList (dict "env" $env "name" .name) }}
```
- **This appends a dictionary of env and name to the $appsList for each application in the current environment.**

6. **Converting the list to JSON**:
```yaml
{{- toJson $appsList }}
```
- **This converts the list of application parameters into a JSON array to be used in the Argo CD application generation.**

## Final Output
- **After running the above template, the generated JSON will look like this:**
```yaml
[
  {"env": "dev", "name": "app-be"},
  {"env": "dev", "name": "app-fe"},
  {"env": "sit", "name": "app-be"},
  {"env": "sit", "name": "app-fe"}
]
```
## Conclusion
This example shows how to use Go templating in Argo CD's ApplicationSet to dynamically generate application parameters based on a structured YAML configuration. By using list and dict functions, we can create dynamic lists of applications that will be used for generating multiple Argo CD applications. This approach can be extended for more complex use cases and can be adjusted to match various environments and application configurations.