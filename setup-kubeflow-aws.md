# https://www.kubeflow.org/docs/distributions/aws/deploy/install-kubeflow/

# ===============================================================
# EKS Cluster Setup
# ===============================================================
# IMPORTANT : AWS_CLUSTER_NAME needs to conform to regular expression pattern: [a-zA-Z][-a-zA-Z0–9]
export AWS_CLUSTER_NAME=mdai-clust-poc-1
export AWS_REGION=us-west-2
export K8S_VERSION=1.18
export EC2_INSTANCE_TYPE_1=inf1.xlarge



# Example with multiple node groups
<!-- cat << EOF > cluster.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${AWS_CLUSTER_NAME}
  version: "${K8S_VERSION}"
  region: ${AWS_REGION}

managedNodeGroups:
  - name: kubeflow-mng-1
    minSize: 0
    maxSize: 5
    desiredCapacity: 4
    instanceType: ${EC2_INSTANCE_TYPE_1}
  - name: kubeflow-mng-2
    minSize: 0
    maxSize: 2
    desiredCapacity: 0
    instanceType: ${EC2_INSTANCE_TYPE_2}
EOF -->


# Create cluster configuration file for use with eksctl
cat << EOF > cluster.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${AWS_CLUSTER_NAME}
  version: "${K8S_VERSION}"
  region: ${AWS_REGION}

managedNodeGroups:
  - name: kubeflow-mng-1
    minSize: 0
    maxSize: 10
    desiredCapacity: 5
    instanceType: ${EC2_INSTANCE_TYPE_1}
EOF

# Create cluster
eksctl create cluster -f cluster.yaml

# To change nodegroup scaling : eksctl scale nodegroup --cluster=CLUSTER_NAME_HERE --nodes=4 ng
# Other example : eksctl scale nodegroup --cluster mdai-cluster-1 --nodes 6 --nodes-max 10 --nodes-min 0 --name kubeflow-mng

# Deploly Kubernetes Dashboard


# ===============================================================


# To Fix AWS access of EKS see : https://aws.amazon.com/premiumsupport/knowledge-center/eks-kubernetes-object-access-error/ 
# Make sure to log in as the user and not the root to see the nodes

# To find aws creds: aws sts get-caller-identity

# ===============================================================
# Kubeflow Setup
# ===============================================================

# Environment Setup

    # Download kfctl 1.2
    wget https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
    # Unpack tar 
    tar -xvf kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
    # Add current directory to path to simplify use of kfctl
    export PATH=$PATH:$PWD

    # Set environment variable for kfctl configuration file (DEX or AWS Cognito)
    export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.2-branch/kfdef/kfctl_aws.v1.2.0.yaml"

    # Create deployment directory for our cluster and change to it
    mkdir ${AWS_CLUSTER_NAME} && cd ${AWS_CLUSTER_NAME}
    # Download the kfctl configuration file
    wget -O kfctl_aws.yaml $CONFIG_URI

# Kubeflow Configuration
    # Setup AWS Role (using AWS IAM Roles for Service Accounts (recommended) or using Node Groupe Role)
    # kfctl will create two roles, kf-admin-{aws-region}-{cluster-name} and kf-user-{aws-region}-{cluster-name}
    # Change password from kfctl_aws.yaml

# Kubeflow Deployment
    
    # Initialize Kubeflow cluster
    kfctl apply -V -f kfctl_aws.yaml

# Wait for all resources to become ready 
kubectl -n kubeflow get all

# Access Kubeflow central dashboard
# Get address by running this command
kubectl get ingress -n istio-system



# ===============================================================

## Install & Setup aws cli

## Install eksctl

## Install Kubeflow prerequisites

https://github.com/kubeflow/manifests/tree/v1.4.0-rc.0#prerequisites

## Setup aws eks cluster by following kubeflow install guide

## Install Kubeflow 

https://github.com/kubeflow/manifests/tree/v1.4.0-rc.0#installation

### If not production then setup portforwarding

kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

## Create dex account by logging in
