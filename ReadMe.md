
---

## ArgoCD Application Management

ArgoCD is used to manage applications in your Kubernetes cluster. Each application is defined in a YAML file located at `argocd-apps/ui/{env}/app-{env}.yaml`. These YAML files contain the necessary configuration to deploy and manage your applications.

## Secret Management with AWS Secret Manager

AWS Secret Manager is utilized for secure secret storage. Secrets are created and managed through the AWS Management Console. ArgoCD connects to AWS Secret Manager to retrieve secrets required by the applications it deploys. This approach ensures sensitive information is kept secure and separate from the application configuration.

To add a new secret to AWS Secret Manager:
1. Navigate to AWS Secret Manager > Store a new secret.
2. Enter the key/value pairs for the secret, provide a name (e.g., `stage/secret/regcred`), and store the secret.

## ArgoCD Plugin Installation

ArgoCD Vault plugin is set up by applying the `argocd-cm.yaml` and `argocd-repo-server.yaml` configuration files. This integration allows ArgoCD to work with HashiCorp Vault for secret management.

## Application Manifest Structure

Each application's configuration is stored in a `.yaml` file within the `cronjobs` and `services` folders. These YAML files include specifications for Deployment, Service, and Istio VirtualService components. Applications can be synchronized manually or automatically based on operational requirements.

## Git Workflow Deployment (Stage Environment)

The stage environment's deployment workflow is triggered by new tags in the Git repository. The CI (Continuous Integration) pipeline builds Docker images tagged with version numbers (`vX.X.X`) and Git commit SHA. Images are then pushed to the `ghcr.io` container registry.

The CD (Continuous Deployment) pipeline updates the image tag in the application's YAML file located in the services folder. A new commit is made to the ArgoCD repository, triggering a webhook that forces an application sync. As a result, ArgoCD updates the deployment with the new image tag.

## Logging and Monitoring

Logging is organized within the `elk` folder. The process involves using Filebeat to collect and forward application logs to Elasticsearch, which can be visualized using Kibana. Configuration details for log indexing can be found in `elk/filebeat/cloud-filebeat-configmap.yaml`.

For monitoring purposes, Prometheus and Grafana are set up. Prometheus collects metrics, and Grafana is used to create custom dashboards for monitoring and alerting.

## Tracing and Observability

The `kiali` folder manages tracing and observability components. Kiali and Jaeger UIs are employed to monitor the service mesh. Kiali provides a dashboard for visualization and management of service mesh traffic, while Jaeger offers insights into application traces.

## Adding Users for EKS Access with SSO

To provide users access to the EKS cluster through Single Sign-On (SSO):
1. Log in to the SSO admin console.
2. Create or add a user and grant access to the appropriate group.
3. Configure AWS CLI SSO profile with the necessary information.
4. Use the AWS CLI to update the kubeconfig for the EKS cluster.

## ArgoCD Installation

ArgoCD is installed by applying the installation YAML file after creating the necessary namespace. Additionally, the ArgoCD UI can be exposed using Istio for external access.

## Istio Installation

Istio is downloaded and installed using the Istio CLI. A specific version is chosen, and the installation profile is set to "demo." A label is applied to the default namespace to enable automatic sidecar proxy injection.

## Integrate EKS with ArgoCD

To integrate your EKS cluster with ArgoCD:
1. Log in to ArgoCD using the CLI.
2. Use the AWS CLI to update the kubeconfig for the EKS cluster.
3. Add the EKS cluster to ArgoCD using the cluster's ARN.

## ArgoCD Auto-Sync for New Secrets

The Reloader tool is used to watch for changes in ConfigMaps and Secrets. When changes are detected, it triggers a rolling upgrade on relevant resources, ensuring that updated configurations take effect.

## Adding New User to ArgoCD

To add a new user to ArgoCD:
1. Access the EKS cluster.
2. Obtain ConfigMap and ArgoCD ConfigMap YAML files.
3. Apply modifications to the YAML files for RBAC and accounts.
4. Apply the modified YAML files to the cluster.
5. Use the ArgoCD CLI to set a password for the new user.

---

This comprehensive explanation encompasses the details you've provided. If you require further assistance or explanations on specific topics, please don't hesitate to ask!
