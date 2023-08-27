# ArgoCD-Application Management and Deployment Guide

## Introduction

This repository contains documentation and guidelines for managing applications using ArgoCD, a declarative, GitOps continuous delivery tool.

## Table of Contents

- [Secret Management](#secret-management)
- [Adding a New AWS Secret Manager](#adding-a-new-aws-secret-manager)
- [ArgoCD Plugin](#argocd-plugin)
- [Application Manifest](#application-manifest)
- [Git Workflow Deployment](#git-workflow-deployment)
- [Logging](#logging)
- [Tracing and Observability](#tracing-and-observability)
- [Monitoring](#monitoring)
- [Adding User Access to EKS with SSO](#adding-user-access-to-eks-with-sso)
- [Accessing the EKS Cluster](#accessing-the-eks-cluster)
- [Installing ArgoCD](#installing-argocd)
- [Installing Istio](#installing-istio)
- [Integrating EKS Cluster with ArgoCD](#integrating-eks-cluster-with-argocd)
- [ArgoCD Auto-Sync for New Secrets](#argocd-auto-sync-for-new-secrets)
- [Adding a New User to ArgoCD](#adding-a-new-user-to-argocd)

## Secret Management

- Secrets are created and managed using AWS Secret Manager.
- ArgoCD connects to AWS Secrets Manager to retrieve secrets for application deployment.

## Adding a New AWS Secret Manager

1. Navigate to AWS Secret Manager and create a new secret.
2. Provide key/value information and follow the wizard.
   ![AWS Secret Manager](images/aws-secret-manager.png)

## ArgoCD Plugin

- Use `argocd-cm.yaml` and `argocd-repo-server.yaml` to install the ArgoCD Vault Plugin.

## Application Manifest

- Application manifests are located in the `cronjobs` and `services` folders.
- Each `.yaml` file contains Deployment, Service, and Istio VirtualService configurations.

## Git Workflow Deployment

- Stage environment deployment is triggered by pushing a new tag in the format `vX.X.X`.
- Continuous Integration (CI) pipeline builds Docker images with tags: "vX.X.X" and "sha-gitcommit".
- Images are pushed to `ghcr.io`.
- Continuous Deployment (CD) pipeline updates the image tag in application YAML files, triggers sync via webhook, and rolls out new deployments.
- Sample pipeline: [GitHub Actions Run](https://github.com/pocketsolutions/argocd-ci-test/runs/8202275389?check_suite_focus=true)
  ![GitHub Actions Pipeline](images/mermaid-diagram.png)

## Logging

- Logging is managed in the `elk` folder.
- Filebeat collects app logs, sends them to Elasticsearch, and visualizes them in Kibana.
- Detailed configuration in `elk/filebeat/cloud-filebeat-configmap.yaml`.

## Tracing and Observability

- Kiali and Jaeger UI for tracing and observability are managed in the `kiali` folder.
- Monitor applications through the Kiali dashboard.

## Monitoring

- Monitoring is managed in the `monitoring` folder.
- Prometheus and Grafana UI are set up for monitoring and alerting.

## Adding User Access to EKS with SSO

1. Access the AWS SSO admin console.
2. Add or create a new user and grant them the appropriate group permissions.
3. Configure AWS CLI SSO using the provided profile example.
4. Log in to EKS as the cluster creator.
5. Update the `aws-auth` ConfigMap in EKS to add the new user's role.

## Accessing the EKS Cluster

- Follow the guide to configure your AWS CLI and access your EKS cluster.

## Installing ArgoCD

1. Ensure you have `kubectl` and a kubeconfig file.
2. Install Argo CD by applying the installation YAMLs.
3. Expose Argo CD using Istio.
4. Access the Argo CD console.

## Installing Istio

1. Download Istio and set the PATH.
2. Install Istio to your EKS cluster.
3. Label namespaces for automatic Envoy sidecar injection.

## Integrating EKS Cluster with ArgoCD

1. Log in to ArgoCD.
2. Log in to your EKS cluster.
3. Add your EKS cluster to ArgoCD.
4. Check the ArgoCD console for the integrated cluster.

## ArgoCD Auto-Sync for New Secrets

- Use [Reloader](https://github.com/stakater/Reloader) to watch ConfigMap and Secret changes and perform rolling upgrades on relevant resources.
- Install Reloader and annotate relevant Deployments.

## Adding a New User to ArgoCD

1. Log in to your EKS cluster.
2. Update the `argocd-rbac-cm` and `argocd-cm` ConfigMaps with new user information.
3. Apply the updated ConfigMaps.
4. Log in to ArgoCD via CLI and create a password for the new user.

---
