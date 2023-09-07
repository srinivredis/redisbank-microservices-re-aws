# Deploying to AWS Elastic Kubernetes Service - EKS

You will deploy bank microservices application in AWS EKS. The redis database will be the fully managed redis database on cloud (AWS).

We will use AWS LoadBalaner Controller to enable ingress.

Once the below EKS set up is complete, you can follow the instructions to create the containers, push them to ECR, and leverage the manifest files to deploy each application.


## Configure your AWS CLI.
```
aws configure
```
This tool will prompt you to enter the following information, to configure your CLI:
- `AWS Access Key ID [None]:`
- `AWS Secret Access Key [None]`
- `Default region name [None]:`
- `Default output format [None]:`

You can get your `AWS Access Key ID` and `AWS Secret Access Key` from the AWS Webconsole.

Go to `IAM` ==> `Security Credentials`

![](images/16a-lambda.png)

![](images/16-lambda.png)

Go to `Access keys` section and create one.

![](images/17-lambda.png)

We are not building a Production system. Ignore this warning by click on `I understand...` and move forward by clicking on `Create access key`

![](images/18-lambda.png)

Once the access key is created, take a note of them from this screen. You will need it to configure your AWS CLI.

![](images/19-lambda.png)

Now feed this information to your `aws configure` tool. Here is a typical example:

> NOTE: Do not execute the following snippet. Its just an example of a typical output.
```
[centos@ip-172-31-9-71 lamdba]$ aws configure
AWS Access Key ID [None]: AKIXXXXXXXXXXFGF
AWS Secret Access Key [None]: 3+QxxxxxxxxXXXXXXXXXxxxxxxXXXXXMvY
Default region name [None]: us-west-2
Default output format [None]:
```

## Install eksctl 

1.  Lets install eksctl. Additional details can be found from [here](https://github.com/eksctl-io/eksctl/blob/main/README.md#installation).

```
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

## Install helm

1. Install Helm. This is neeed for installing AWS Ingress controller.
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm version
```

## Install kubectl 

1.  Install kubectl. We will install 1.25 version for this lab. Verify the downloaded binary with the SHA-256 checksum for your binary. Additional details related to K8s releases can be found from [here](https://kubernetes.io/releases/). There are newer versions 1.26, 1.27 available now.

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.9/2023-05-11/bin/linux/amd64/kubectl
```

2. Verify the downloaded binary with the SHA-256 checksum for your binary. Download the SHA-256 checksum for your cluster's Kubernetes version from Amazon S3.

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.9/2023-05-11/bin/linux/amd64/kubectl.sha256
```
3. Check the SHA-256 checksum for your downloaded binary with one of the following commands. The output should be "kubectl: OK"
```
sha256sum -c kubectl.sha256
```
4. Apply execute permissions to the binary. Copy the binary to a folder in your PATH. Validate by running the version command 

```
chmod +x ./kubectl

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

kubectl version --short --client

```

## Deploy EKS cluster 

Now we are ready to deploy our EKS cluster. Give a cluster name 

```
eksctl create cluster <your cluster number>
```

If you are root or IAM user with "Admin Access" then the cluster will be created in about 15 to 20 minutes.


## Deploy AWS Ingress Controller
1. Create IAM Policy
    ```
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

     aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
    ```

You can see the policy on the web console also.

2. You can use eksctl or the AWS CLI and kubectl to create the IAM role and Kubernetes service account. For this you need to associate an OIDC provided with the cluster. Lets do that first.

  ```
  export cluster_name=<your cluster name>
  oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
  eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
  ```

3. Now create the IAM role. Change the ARN of the policy to your ARN. You can get this from web console also under IAM.   This will take 1-2 minutes.
Please edit cluster name  with your cluster name

  ![](images/arn-search.png)
  ![](images/find-arn.png)


  ```
  eksctl create iamserviceaccount \
  --cluster=<your cluster name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=<your policy ARN> \
  --approve
  ```
4. Helm update before installing the AWS ingress controller
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

5. Install controller. Please edit cluster name  with your cluster name
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<Your cluster Name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

6. You will see output like this 

```
NAME: aws-load-balancer-controller
LAST DEPLOYED: Tue Aug 15 15:43:34 2023
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!
```

7. Validate to see you get the below output
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           2m54s
```

Now you can deploy your apps in this cluster.


You have a lab around this at this [location](https://github.com/srinivredis/hands-on-labs-cloud-native-modern-app)

