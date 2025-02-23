# Network Considerations
I have used AWS with EKS with managed worker nodes.

## VPC with public and private endpoints
An EKS cluster consists of 2 VPCs. The first VPC is managed by AWS where the Kubernetes Control Plane resides within this VPC (this cannot be seen by the users). The second VPC is the customer VPC which we specify during the cluster creation. This is where we place all the worker nodes.

### Cluster Endpoint Access Type
This option allows the public endpoint as explained above, but the Customer Managed VPC traffic (eg: worker nodes trying to connect to the EKS control plane) will go through the EKS-managed Elastic Network Interface (ENI) through a private endpoint.
This situation is ideal if you’d like to allow your cluster to be accessible via the internet, but you’d like to allow your worker nodes to be in a private subnet and communicate with the EKS control plane through a private endpoint.

### Public and Private Subnets
This is the widely used VPC setup for EKS, where the worker nodes reside within the private subnets and the NAT Gateway and load balancers are placed within the public subnet.

### NAT Gateway
You can use a network address translation (NAT) gateway to enable instances in a private subnet to connect to the internet or other AWS services, but prevent the internet from initiating a connection with those instances.In order to access internet to your private subnet, NAT Gateway must be added to Public Subnet only.

### Route table 
It contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed. To put it simply, a route table tells network packets which way they need to go to get to their destination.


## Deploying EKS with eksctl
eksctl is an open source tool to quickly setup a fully functional EKS cluster with minimal effort. 
```
eksctl create cluster --name mycluster --region eu-central-1
```  
Once the project is complete, delete the cluster
```
eksctl delete cluster --name mycluster --region eu-central-1
```

## Configure IAM OIDC Provider
```
export cluster_name=mycluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

## Deploy ALB controller
Selecting the Application Load Balancer (ALB) as the workload is HTTP/HTTPS.The ALB is deployed in the public subnets and is accessible from the internet.The ALB routes traffic to the private web servers in the private subnets.The private web servers are not directly accessible from the internet, but the ALB acts as a secure gateway.
By default the AWS Load Balancer controller will register targets using "Instance" type and this target will be the Worker Node’s IP and NodePort, implying traffic from the Load Balancer will be forwarded to the Worker Node on the NodePort.

Advanced features of the ALB:
- Path based routing
- Host based routing
- SSL/TLS termination
- Health checks
- Sticky sessions

### Setup alb add on
Download IAM Policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
Create IAM Role
```
eksctl create iamserviceaccount \
  --cluster=mycluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

### Install AWS Load Balancer Controller
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
Verify the deployments are running
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

## EBS CSI Plugin configuration
The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.

Create an IAM role and attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. You can create an IAM role and attach the AWS managed policy with the following command. The command deploys an AWS CloudFormation stack that creates an IAM role and attaches the IAM policy to it.

```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <YOUR-CLUSTER-NAME> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```

```
eksctl create addon --name aws-ebs-csi-driver --cluster <YOUR-CLUSTER-NAME> --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

