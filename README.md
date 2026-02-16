# Lesson 8–9: Jenkins + Terraform + Helm + Argo CD (GitOps CI/CD)

## Project Overview
This project implements a complete CI/CD pipeline using Terraform, Jenkins, Helm, Argo CD and AWS (EKS + ECR).

The goal of the project is to demonstrate a modern GitOps-based delivery workflow, where infrastructure is provisioned declaratively using Terraform, Continuous Integration is handled by Jenkins, Continuous Deployment is handled by Argo CD, and application deployments are fully automated and driven by Git.

## Architecture Overview

Infrastructure:
- Terraform remote backend using S3 and DynamoDB
- VPC with public and private subnets and NAT Gateway
- EKS cluster with managed node group
- ECR repository for Docker images

CI/CD:
- Jenkins installed on EKS using Helm
- Jenkins pipeline builds Docker image, pushes it to ECR and updates Helm chart values in Git
- Argo CD installed on EKS using Helm
- Argo CD Application automatically synchronises changes from Git to the cluster (GitOps)

## Repository Structure

Infrastructure repository:
lesson-8-9-infra/
- backend.tf
- providers.tf
- k8s_providers.tf
- main.tf
- variables.tf
- outputs.tf
- modules/
  - s3-backend
  - vpc
  - ecr
  - eks
  - jenkins
  - argo_cd
- README.md

Helm charts repository:
lesson-8-9-charts/
- charts/
  - django-app

## How to Apply Terraform

Prerequisites:
- AWS CLI configured using aws configure
- Terraform version 1.5 or newer
- AWS account with sufficient permissions

Steps:
1. Navigate to the infrastructure repository:
   cd lesson-8-9-infra
2. Initialise Terraform:
   terraform init
3. Apply the configuration:
   terraform apply

Terraform creates the AWS infrastructure, installs Jenkins and Argo CD using Helm, and creates an Argo CD Application for GitOps-based deployment.

## How to Test the Jenkins Job

1. After terraform apply, Jenkins is exposed via a Kubernetes LoadBalancer service.
2. Retrieve the Jenkins administrator password using:
   terraform output jenkins_admin_password
3. Log in to the Jenkins UI.
4. Run the pipeline which:
   - builds the Docker image,
   - pushes the image to Amazon ECR,
   - updates the Helm chart image tag in the Git repository.

## How to View the Result in Argo CD

1. Argo CD is exposed via a Kubernetes LoadBalancer service.
2. Retrieve the initial Argo CD admin password using:
   terraform output argocd_initial_admin_password
3. Log in to the Argo CD UI.
4. Observe the django-app Application which tracks the Helm chart repository.
5. After Jenkins updates the Helm values in Git, Argo CD automatically synchronises and deploys the changes to the cluster.

## Known Limitations

During local testing, access to the Kubernetes API using kubectl or Terraform Kubernetes and Helm providers may fail due to local DNS resolution issues for *.eks.amazonaws.com, depending on router or network configuration.

The infrastructure itself is successfully created:
- EKS cluster status is ACTIVE
- Managed node group is ACTIVE
- Jenkins and Argo CD are correctly defined and installed via Terraform and Helm

In production or CI environments, or when using public DNS resolvers such as 1.1.1.1, the setup works as expected.

## Cleanup

To avoid unnecessary cloud costs, all resources can be removed using:
terraform destroy

This also removes the S3 bucket and DynamoDB table used for Terraform state storage.

## Summary

This project demonstrates Infrastructure as Code with Terraform, CI with Jenkins, CD with Argo CD, and a GitOps deployment model on AWS EKS. It reflects how modern DevOps teams deliver changes in a fast, stable and predictable way.
