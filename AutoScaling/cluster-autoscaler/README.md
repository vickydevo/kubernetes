# cluster-autoscaler

## Pre-Requisites

```shell
EKS-Cluster Setup
```

## Clone the cluster autoscaler code

```shell
git clone https://github.com/Naresh240/kubernetes.git
cd AutoScaling/cluster-autoscaler
```

## Install service-account

```shell
kubectl apply -f service-account.yml
```

## Create IRSA for cluster autoscaler to communicate Auto scaling service in AWS

1. Create policy with below json file
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions",
                "ec2:DescribeInstanceTypes"
            ],
            "Resource": "*"
        }
    ]
}
```

   ```shell
   aws iam create-policy \
    --policy-name AutoScalerIAMPolicy \
    --policy-document file://cluster-autoscaler.json
   ```

2. Create IRSA

```bash
eksctl create iamserviceaccount \
   --cluster=eksdemo \
   --region us-east-1 \
   --namespace=kube-system \
   --name=cluster-autoscaler \
   --role-name AutoscalerAole \
   --attach-policy-arn=arn:aws:iam::400095111010:policy/AutoScalerIAMPolicy \
   --override-existing-serviceaccounts \
   --approve
```

## Run Cluster Autoscaler yaml file

```bash
kubectl apply -f cluster-autoscaler-deployment.yml
```

## Check cluster autoscaler pod

```bash
kubectl get pod -n kube-system
```

## Deploy "ngix" application

```bash
kubectl apply -f deployment.yml
```

## Cluster Scale UP: Scale our application to 30 pods

```bash
kubectl scale --replicas=30 deploy ca-demo-deployment
```

## Terminal - 2: Verify nodes

```bash
kubectl get nodes -o wide
```

## Cluster Scale DOWN: Scale our application to 1 pod
  Terminal - 1: Keep monitoring cluster autoscaler logs
	  
    kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
  
  Terminal - 2: Scale down the demo application to 1 pod
	  
    kubectl scale --replicas=1 deploy ca-demo-deployment 
	
  Terminal - 2: Verify nodes
	
    kubectl get nodes -o wide    
