# ArgoCD-Application Management and Deployment Guide

This README document provides comprehensive information on managing applications using ArgoCD, integrating with AWS Secret Manager, configuring Kubernetes clusters, implementing GitOps workflows, logging, tracing, observability, and more.

## Table of Contents
- [Application Management](#application-management)
- [Secret Management with AWS Secret Manager](#secret-management-with-aws-secret-manager)
- [Installing ArgoCD Plugins](#installing-argocd-plugins)
- [Application Manifest Structure](#application-manifest-structure)
- [Git Workflow Deployment](#git-workflow-deployment)
- [Logging Setup](#logging-setup)
- [Tracing and Observability](#tracing-and-observability)
- [Monitoring](#monitoring)
- [EKS Access and User Management](#eks-access-and-user-management)
- [Installing ArgoCD](#installing-argocd)
- [Installing Istio](#installing-istio)
- [Integrating EKS Cluster with ArgoCD](#integrating-eks-cluster-with-argocd)
- [Auto-sync with Reloader](#auto-sync-with-reloader)
- [Adding New User to ArgoCD](#adding-new-user-to-argocd)

---

## Application Management

All ArgoCD applications are managed through the UI at [https://argocd.pocketlaw.io/applications](https://argocd.pocketlaw.io/applications). The application definitions are located at `argocd-apps/ui/{env}/app-{env}.yaml`. Each application has a unique source path that calls the `.yaml` file in the services folder.

## Secret Management with AWS Secret Manager

- Secrets are created and managed using AWS Secret Manager via the console.
- ArgoCD connects to AWS Secrets Manager to retrieve secrets and run applications.
- Install the ArgoCD plugin to establish the connection between ArgoCD and AWS Secrets Manager.

### Adding a New AWS Secret Manager

1. Go to AWS Secret Manager > Store a new secret.
2. Input your key/value and proceed.
3. Fill in the secret name (e.g., `stage/secret/regcred`) and store the secret.

## Installing ArgoCD Plugins

ArgoCD plugins are applied using `argocd-cm.yaml` and `argocd-repo-server.yaml` to install the ArgoCD Vault plugin.

## Application Manifest Structure

Each `.yaml` file in the cronjobs and services folders contains Deployment, Service, and Istio VirtualService definitions to expose services externally. Applications can be synchronized manually or automatically based on demand and strategy.

## Git Workflow Deployment

- Staging environment triggers build and deployment on new tag `v*.*.*`.
- CI pipeline:
  - Docker image is built with tags: "vX.X.X" and "sha-gitcommit".
  - Images are pushed to `ghcr.io`.
- CD pipeline:
  - Clone the ArgoCD repo and update the new image tag in application yaml files in the services folder.
  - After changing the Docker image tag, create a new commit and push to ArgoCD.
  - A webhook triggers the ArgoCD API to force application sync.
  - ArgoCD automatically syncs and rolls out new deployments with the new image tag.
- Sample pipeline: [GitHub Actions Run](https://github.com/pocketsolutions/argocd-ci-test/runs/8202275389?check_suite_focus=true)

## Logging Setup

- Logs are managed in the `elk` folder.
- Filebeat collects and sends app logs to Elasticsearch, which can then be visualized on Kibana.
- Index configuration details are available at `elk/filebeat/cloud-filebeat-configmap.yaml`.

## Tracing and Observability

- Tracing and observability are managed in the `kiali` folder.
- Monitoring can be done via the Kiali dashboard.

## Monitoring

- Monitoring is managed in the `monitoring` folder.
- Prometheus and Grafana UI are set up to add dashboards for monitoring and alerting.

## EKS Access and User Management

- Detailed steps for adding users to access EKS with SSO are outlined in the documentation.
- The process involves configuring AWS CLI SSO and updating the `configmap` in EKS for user access.

## Installing ArgoCD

Follow the [ArgoCD installation guide](https://argo-cd.readthedocs.io/en/stable/getting_started/) to set up ArgoCD. Ensure you meet the requirements, have a valid `kubeconfig` file, and have CoreDNS installed.

## Installing Istio

- Download Istio using: `curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.15.3 TARGET_ARCH=x86_64 sh -`.
- Install Istio using the provided guide. Make sure to label the default namespace for automatic Envoy sidecar injection.

## Integrating EKS Cluster with ArgoCD

1. Login to ArgoCD: `argocd login argocd.pocketlaw.io`.
2. Login to your EKS cluster: `aws eks --region eu-north-1 update-kubeconfig --name eks-dev`.
3. Add the cluster to ArgoCD: `argocd cluster add arn:aws:eks:eu-north-1:381422252640:cluster/eks-dev`.
4. Check the ArgoCD console for the added cluster.

## Auto-sync with Reloader

- Use [Reloader](https://github.com/stakater/Reloader) to watch changes in ConfigMaps and Secrets and trigger rolling upgrades on relevant resources.
- Install Reloader: `kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml`.
- Add the annotation `reloader.stakater.com/auto: "true"` to relevant Deployment `.yaml` files.

## Adding New User to ArgoCD

1. Login to EKS cluster.
2. In the argocd namespace, update the `argocd-rbac-cm` and `argocd-cm` ConfigMaps to add the new user.
3. Use the ArgoCD CLI to create a password for the new user.

---

This optimized README provides clear and organized instructions for managing applications, integrating with AWS services, setting up ArgoCD, configuring Kubernetes clusters, and more. It is designed to improve readability and usability for users working with these technologies.
