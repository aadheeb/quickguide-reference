## To create a Kubernetes cluster using eksctl

**To Create a cluster you need to install `eksctl`**

```
- curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
- sudo mv /tmp/eksctl /usr/local/bin
- export PATH=$PATH:/usr/local/bin
- echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
```
-To verify the installation:
    `eksctl version`

**Create an IAM user configured with the following permissions.**

```
- AmazonEC2FullAccess (AWS Managed Policy)
____________________________________________

  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:*",
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "autoscaling:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": [
                        "autoscaling.amazonaws.com",
                        "ec2scheduled.amazonaws.com",
                        "elasticloadbalancing.amazonaws.com",
                        "spot.amazonaws.com",
                        "spotfleet.amazonaws.com",
                        "transitgateway.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
```
```
	- AWSCloudFormationFullAccess (AWS Managed Policy)
  _____________________________________________________


  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*"
            ],
            "Resource": "*"
        }
    ]
}
```

```
	- EksAllAccess(customcreated)
  ______________________________

  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        },
        {
            "Action": [
                "ssm:GetParameter",
                "ssm:GetParameters"
            ],
            "Resource": [
                "arn:aws:ssm:*:<account_id>:parameter/aws/*",
                "arn:aws:ssm:*::parameter/aws/*"
            ],
            "Effect": "Allow"
        },
        {
             "Action": [
               "kms:CreateGrant",
               "kms:DescribeKey"
             ],
             "Resource": "*",
             "Effect": "Allow"
        },
        {
             "Action": [
               "logs:PutRetentionPolicy"
             ],
             "Resource": "*",
             "Effect": "Allow"
        }        
    ]
}
```

	- IamLimitedAccess(customcreated)


Reference-link: `https://eksctl.io/usage/minimum-iam-policies/`


**We can create a cluster with a single command or with an yaml file**

 -To Create a Cluster run this command
 
     `eksctl create cluster --name my-cluster --region region-code --version 1.24 --vpc-private-subnets subnet-ExampleID1,subnet-ExampleID2 --without-nodegroup`

 -Create using a cluster.yaml file
 	
  First, create a "cluster.yaml" file
		
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: unitear-prod (custom-name)
  region: us-west-2
  version: "1.24"

iam:
  withOIDC: true

nodeGroups:
  - name: ng-1
    instanceType: t3a.medium
    desiredCapacity: 3
    ssh:
      publicKeyName: unitear-prod-ng-key

availabilityZones: ['us-west-2a', 'us-west-2b', 'us-west-2c'] 
```

 -Then, run this command:
 
      `eksctl create cluster -f cluster.yaml`

**Ingress Controller for ALB(AWS Application Load Balancer)**

 -Create an IAM policy Reference: 
```
https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/iam-policy.json
```

 -Then, Create K8s service account using eksctl:
 
```
eksctl create iamserviceaccount --name alb-ingress-controller --namespace kube-system --cluster unitear-prod --attach-policy-arn arn:aws:iam::165374390891:policy/AWSLoadBalancerControllerIAMPolicy --approve
```


- This eksctl command will deploy a new CloudFormation stack with an IAM role. Wait for it to finish before keep executing the next steps.


**Install the TargetGroupBinding CRDs**

 -Step-1
    `kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"`
    
 -Step-2 (verify)
    `kubectl get crd`


**Prerequisites**

- We have to verify if the AWS Load Balancer Controller version has been set

```
if [ ! -x ${LBC_VERSION} ]
  then
    tput setaf 2; echo '${LBC_VERSION} has been set.'
  else
    tput setaf 1;echo '${LBC_VERSION} has NOT been set.'
fi
```

- If the result is ${LBC_VERSION} has NOT been set.,

  Refer here: `https://archive.eksworkshop.com/020_prerequisites/k8stools/#set-the-aws-load-balancer-controller-version`




**Deploy the Helm chart**

- Make sure that the helm is installed.

- `helm repo add eks https://aws.github.io/eks-charts`

- `helm upgrade -i aws-load-balancer-controller \
    eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=unitear-prod(custom-name) \
    --set serviceAccount.create=false \
    --set serviceAccount.name=alb-ingress-controller \
    --set image.tag="${LBC_VERSION}" \
    --version="${LBC_CHART_VERSION}"`

- `kubectl -n kube-system rollout status deployment alb-ingress-controller`





Reference:
`https://archive.eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/`








