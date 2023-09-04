# ArgoCD Application Management

This repository contains guidelines and information for managing applications using ArgoCD, integrating with AWS services, managing secrets, and configuring various components.

## Folder Structure

- All ArgoCD applications are managed through the UI at [https://argocd.pocketlaw.io/applications](https://argocd.pocketlaw.io/applications) and are stored in the `argocd-apps/ui/{env}/app-{env}.yaml` directory.
- Each application references a `.yaml` file located in the services folder, specifying its source path.

## Secret Management with AWS Secret Manager

- Secrets are created and managed in AWS Secret Manager through the console.
- ArgoCD connects to AWS Secrets Manager to retrieve secrets for application execution.
- Install the ArgoCD plugin to enable integration between ArgoCD and AWS Secrets Manager.

### Adding a New AWS Secret Manager Secret

1. Go to AWS Secret Manager > Store a new secret.
2. Enter your key/value pairs and proceed.
3. Specify a secret name (e.g., `stage/secret/regcred`), and complete the setup.
<img width="1815" alt="Screen Shot 2022-10-10 at 16 53 34" src="https://user-images.githubusercontent.com/74416058/194840337-61f5a3ea-1fac-468d-8e65-995935edf9cf.png">

## ArgoCD Plugin Integration

- Apply `argocd-cm.yaml` and `argocd-repo-server.yaml` to install the ArgoCD Vault Plugin.

## Application Manifest Structure

- Each `.yaml` file in the cronjobs and services folders contains a Deployment, Service, and Istio VirtualService to expose the service externally.
- Applications on ArgoCD can be manually or automatically synchronized based on demand and strategy.

## Git Workflow Deployment for Stage Environment

- Stage environment deployment is triggered by pushing new tags of the format `v*.*.*`.
- CI Pipeline:
  - Builds Docker images with tags: `vX.X.X` and `sha-gitcommit`.
  - Pushes built images to `ghcr.io`.
- CD Pipeline:
  - Updates the new image tag in the application YAML file in the services folder.
  - Creates a commit to push changes to ArgoCD's repository.
  - A webhook triggers ArgoCD's API, forcing application synchronization.
  - ArgoCD automatically syncs and rolls out a new deployment with the updated image tag.
- See a sample pipeline run [here](https://github.com/pocketsolutions/argocd-ci-test/runs/8202275389?check_suite_focus=true
  ![alt text](mermaid-diagram-2022-09-01-130650.png).

## Logging with ELK Stack

- Logs are managed in the `elk` folder.
- Filebeat collects and forwards application logs to Elasticsearch.
- Logs are visualized using Kibana.
- Detailed index configuration is available in `elk/filebeat/cloud-filebeat-configmap.yaml`.

## Tracing and Observability with Kiali and Jaeger UI

- Tracing and observability components are managed in the `kiali` folder.
- Monitoring is done through the Kiali dashboard.

## Monitoring Setup

- Monitoring tools are configured in the `monitoring` folder.
- Prometheus and Grafana are set up to provide monitoring and alerting dashboards.

## Adding Users for EKS Access with SSO

1. Access the AWS SSO admin console [here](https://eu-north-1.console.aws.amazon.com/singlesignon/identity/home?region=eu-north-1#!/dashboard).
2. Add or create a new user (e.g., `devops-user`) and grant them the "EKS_Cluster_Admins" group.
3. Configure AWS CLI SSO
4. Login to EKS as cluster creator:
  `- aws eks --region eu-north-1 update-kubeconfig --name pl-eks-stage-en1 --profile <cluster_creator>`
5. Update configmap in EKS
  - kubectl edit configmap aws-auth -n kube-system. Add following lines:
```
    - groups:
      - system:masters
      rolearn: arn:aws:iam::381422252640:role/AWSReservedSSO_EKSClusterAdminAccess_427e09b594653102
      username: devops-user
```
- Step 6: Test access to EKS with new user:
```
    - aws eks --region eu-north-1 update-kubeconfig --name pl-eks-stage-en1 --profile <devops-user>
```
## EKS Cluster Access

- Detailed steps for accessing the EKS cluster are provided in the [Amazon EKS documentation](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/).
- aws eks --region eu-north-1 update-kubeconfig --name pl-eks-stage-en1 --profile <profile_name_sso>

## Installing ArgoCD

### Requirements
- Installed `kubectl` command-line tool.
- Have a kubeconfig file (default location is `~/.kube/config`).
- CoreDNS should be enabled (e.g., for MicroK8s: `microk8s enable dns && microk8s stop && microk8s start`).

### Installation Steps
1. Access your EKS cluster.
2. Create the `argocd` namespace: `kubectl create namespace argocd`.
3. Apply the ArgoCD installation YAML: `kubectl apply -n argocd -f ./argocd-install/install.yaml`.
4. Expose ArgoCD for external access using Istio:
   - Edit the `.yaml` content in `services/istio-gw-vs` as necessary.
   - Apply the Istio gateway and virtual service configurations.

## Installing Istio

1. Download Istio: `curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.15.3 TARGET_ARCH=x86_64 sh -`.
2. Access the existing Istio configuration at `istio-ingress/<env>`.
3. Export the Istio `bin` folder to the `PATH`: `export PATH=$PWD/bin:$PATH`.
4. Install Istio on your EKS cluster and enable automatic Envoy sidecar injection.
```
    $ kubectl label namespace default istio-injection=enabled
```

## Integrating EKS Cluster with ArgoCD

1. Login to ArgoCD: `argocd login argocd.pocketlaw.io`.
2. Login to your EKS cluster: `aws eks --region eu-north-1 update-kubeconfig --name eks-dev`.
3. Add your EKS cluster to ArgoCD: `argocd cluster add arn:aws:eks:eu-north-1:381422252640:cluster/eks-dev`.
4. Check on ArgoCD console
![alt text](argocd-connect-eks.png)

## ArgoCD Auto-Sync with New Secrets

- Utilize the [Reloader](https://github.com/stakater/Reloader) tool to watch for changes in ConfigMaps and Secrets and perform rolling upgrades.
- Install Reloader: `kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml`.
- Add the `reloader.stakater.com/auto: "true"` annotation to deployment `.yaml` files.

## Adding a New User to ArgoCD

1. Log in to your EKS cluster.
2. Update the `argocd-rbac-cm` and `argocd-cm` ConfigMaps with new user information.
   - kubectl get cm argocd-rbac-cm -o yaml > argocd-rbac-cm-xxxx.yaml (add oneline: ewinberg, role:backend\n")
   - kubectl get cm argocd-cm -o yaml > argocd-cm-yyyy.yaml (Add one line: accounts.ewinberg: apiKey,login)
3. Apply the updated ConfigMaps.
   - kubectl apply -f argocd-rbac-cm-xxxx.yaml argocd-cm-yyyy.yaml
4. Log in to ArgoCD via CLI and create a password for the new user.
   - argocd login argocd.pocketlaw.io
   - argocd account update-password --account <your_new_user> --new-password <your_new_password>
