# Deploy an EKS cluster with eksctl

To test this sample repository you need an Amazon EKS cluster with [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.3/) installed. Follow the instructions below to create a new cluster.

**NOTE:** This sample creates an EKS cluster with 2 m6i.large nodes. The total cost of running the infrastructure for this sample in us-west-2 is approximately $0.40 / hour (considering EKS control plane, EC2 instances, Network Load Balancer and NAT Gateway costs). Remember to [delete](../README#clean-up) the cluster once you're finished testing.

## Prerequisites:

1. Download and install eksctl. You can find instructions [here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html).

1. Download and install the AWS CLI. You can find instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

## Create the cluster

1. Create an IAM Policy to grant permissions to AWS Load Balancer Controller to create and manage Load Balancers. We will use eksctl later to create an IAM Role for the aws-load-balancer-controller Service account.

    ```bash
    curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
    aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam_policy.json
    ```

    **IMPORTANT NOTE:** The above IAM policies contain permissive configuration for `ec2:AuthorizeSecurityGroupIngress` and `ec2:RevokeSecurityGroupIngress`. Follow instructions [here](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/#setup-iam-role-for-service-accounts) so scope it down.

1. Clone this GitHub repository and change to the repository directory.

    ```bash
    git clone https://github.com/aws-samples/flux-eks-gitops-config.git
    cd flux-eks-gitops-config
    ```

1. Within `docs/examples/cluster.yaml`, in the `iam:` section, we're defining an IAM Role for service account aws-load-balancer-controller in the kube-system namespace. Update line 24 with the IAM policy ARN of the policy you've created in the prior step.

    ```yaml
    arn:aws:iam::(your_aws_account_number_here):policy/AWSLoadBalancerControllerIAMPolicy
    ```

1. Create the EKS cluster running the following command. It will take 15-20 minutes to create the cluster.

    ```bash
    eksctl create cluster -f docs/examples/eks-cluster/cluster.yaml
    ```

You're all set. You can now go to the ![deploy section](../README.md#deploy-this-sample).

```bash
eksctl create iamserviceaccount \
--cluster=eks-2023 \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::385379752235:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region eu-central-1 \
--approve
```
