# ArgoCD-Application

This README provides an overview of the ArgoCD-Application project, its features, setup instructions, and usage guidelines.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Secret Management](#secret-management)
- [Git Workflow Deployment](#git-workflow-deployment)
- [Logging](#logging)
- [Tracing and Observability](#tracing-and-observability)
- [Monitoring](#monitoring)
- [Adding Users and Access](#adding-users-and-access)
- [Install ArgoCD](#install-argocd)
- [Install Istio](#install-istio)
- [Integrate with ArgoCD](#integrate-with-argocd)
- [ArgoCD Auto-Sync](#argocd-auto-sync)
- [Adding New Users to ArgoCD](#adding-new-users-to-argocd)

## Introduction

ArgoCD-Application is a project designed to streamline application management using ArgoCD in conjunction with AWS services. It aims to simplify deployment, secret management, and monitoring processes.

## Features

- Centralized management of ArgoCD applications.
- Integration with AWS Secret Manager for secure secret storage.
- Git workflow for deployment to different environments.
- Automated deployment with CI/CD pipelines.
- Logging, tracing, and observability tools.
- Monitoring and alerting using Prometheus and Grafana.

## Installation

To set up the project, follow these steps:

1. Clone this repository: `git clone https://github.com/yourusername/argo-cd-application.git`
2. Install necessary dependencies: `npm install` or as required.
3. Configure AWS Secret Manager and ArgoCD integration.

## Usage

1. Access the ArgoCD UI: https://argocd.pocketlaw.io/applications
2. Manage applications in the `argocd-apps/ui/{env}/app-{env}.yaml` files.
3. Configure source paths for each application.

## Secret Management

- Use AWS Secret Manager for secrets.
- Create secrets via the AWS Console.
- Configure ArgoCD to access secrets for deployments.

## Git Workflow Deployment

- Automated deployments to staging triggered by tag.
- CI pipeline: Docker image build, push to ghcr.io.
- CD pipeline: Update image tag, trigger ArgoCD sync.

## Logging

- Manage logs using the ELK stack.
- Filebeat collects logs, Elasticsearch indexes, Kibana visualizes.

## Tracing and Observability

- Monitor using Kiali and Jaeger UI.
- Detailed setup and management in the `kiali` folder.

## Monitoring

- Setup Prometheus and Grafana for monitoring and alerting.
- Dashboards available in the `monitoring` folder.

## Adding Users and Access

- Add users to EKS with SSO for secure access.
- Configure AWS CLI SSO for user access.

## Install ArgoCD

- Install ArgoCD on EKS cluster.
- Expose ArgoCD UI for external access.

## Install Istio

- Download and install Istio for EKS.
- Configure Istio for automatic sidecar injection.

## Integrate with ArgoCD

- Connect EKS cluster to ArgoCD for application management.
- Use the ArgoCD UI for seamless integration.

## ArgoCD Auto-Sync

- Use Reloader to automatically sync changes.
- Integration instructions provided in the README.

## Adding New Users to ArgoCD

- Manage ArgoCD users and access via kubectl commands.
- Detailed steps outlined in the README.
