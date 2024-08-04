# AWS EKS Setup and Deployment of "2048"

This guide will walk you through setting up an AWS EKS cluster using the `eksctl` command-line utility and deploying the game "2048".

## Prerequisites

Before you begin, ensure you have the following installed and configured:

- **AWS CLI**: The AWS Command Line Interface should be installed and configured with your user account.
- **IAM Permissions**: Ensure your AWS user account has the necessary IAM permissions to create and manage EKS resources.
- **eksctl**: The `eksctl` command-line tool must be installed.
- **kubectl**: The Kubernetes CLI tool `kubectl` should be installed.
- **helm**: The Helm package manager must be installed.

## Setup Instructions

### 1. Create the EKS Cluster

Run the following command to create your EKS cluster:

```bash
eksctl create cluster --name <clusterName> \
--region <regionName> \
--nodegroup-name <nodeGroupName> \
--node-type <instanceType> \
--nodes <numberOfNodes> \
--nodes-min <minimumNumberOfNodes> \
--nodes-max <maximumNumberOfNodes>
```

**Example:**

```bash
eksctl create cluster --name demo-game-2048 \
--region us-east-1 \
--nodegroup-name game-2048-nodes \
--node-type t2.large \
--nodes 2 \
--nodes-min 2 \
--nodes-max 4
```

The cluster creation process may take up to 15 minutes.

### 2. Update `kubectl` Configuration

Once the cluster is ready, update your `kubectl` configuration to point to the new cluster:

```bash
aws eks update-kubeconfig --name <clusterName> \
--region <clusterRegion>
```

**Example:**

```bash
aws eks update-kubeconfig --name demo-game-2048 \
--region us-east-1
```

### 3. Apply Deployment Configuration

Apply the deployment configuration, namespace, service, and ingress resources from the `full_2048.yaml` file:

```bash
kubectl apply -f <fileLocationwithFileName>
```

**Example:**

```bash
kubectl apply -f ./full_2048.yaml
```

### 4. Associate IAM OIDC Provider with the Cluster

Associate an IAM OIDC provider with your cluster to allow it to access AWS resources (e.g., AWS ALB):

```bash
eksctl utils associate-iam-oidc-provider --name <clusterName> \
--region <clusterRegion> \
--approve
```

**Example:**

```bash
eksctl utils associate-iam-oidc-provider --name demo-game-2048 \
--region us-east-1 \
--approve
```

### 5. Create IAM Policy

Create an IAM policy using the provided JSON file:

```bash
aws iam create-policy --policy-name <policyName> \
--policy-document <urlOrFileLocation>
```

**Example (assuming the policy file is downloaded):**

```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerPolicy \
--policy-document file://iam_policy.json
```

### 6. Create IAM Service Account and Attach Policy

Create an IAM service account and attach the IAM policy created in the previous step:

```bash
eksctl create iamserviceaccount \
--cluster <clusterName> \
--region <clusterRegion> \
--namespace kube-system \
--name <serviceAccountName> \
--role-name <roleName> \
--attach-policy-arn arn:aws:iam::<yourAccountID>:policy/<policyName> \
--approve
```

**Example:**

```bash
eksctl create iamserviceaccount \
--cluster demo-game-2048 \
--region us-east-1 \
--namespace kube-system \
--name aws-load-balancer-controller \
--role-name AWSLoadBalancerControllerRole \
--attach-policy-arn arn:aws:iam::2222111333:policy/AWSLoadBalancerControllerPolicy \
--approve
```

### 7. Install AWS Load Balancer Controller

Add the Helm repository and install the AWS Load Balancer Controller:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=<clusterName> \
--set serviceAccount.create=false \
--set serviceAccount.name=<serviceAccountName>
```

**Example:**

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=demo-game-2048 \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```

To get the public accessible URL, run:

```bash
kubectl get ingress -n game-2048
```

---

That's it! Your AWS EKS cluster is now set up, and the game "2048" is deployed.

If you have any questions or run into issues, feel free to ask me on

Linkedin:https://www.linkedin.com/in/abdulmuhd-dev/
