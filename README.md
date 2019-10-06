# eksClusterSetup
Pre-Requisites:
 
make sure to install and configure Kubectl, awscli


$ aws --version
aws-cli/1.16.253 Python/3.7.3 Linux/5.0.0-29-generic botocore/1.12.243

$ aws configure
AWS Access Key ID [None]: xyz
AWS Secret Access Key [None]: xyxz
Default region name [None]: us-east-1
Default output format [None]: json


############EKS Cluster Creation##########
Launch EKS cluster with new VPC: ( edit the file to change CIDR values)

$aws cloudformation create-stack --stack-name EKScluster --template-body file://eksCluster.yml --capabilities CAPABILITY_IAM


To access EKS cluster run below command 

$ aws eks --region region_code update-kubeconfig --name eks_cluster_name

Example:
$ aws eks --region us-east-1 update-kubeconfig --name EKScluster
Added new context arn:aws:eks:us-east-1:1234:cluster/stageCluster to /home/jay/.kube/config

########### Worker Nodes Creation ##############
Copy vpc_ID , subnet_IDs and SecurityGroupIDs from eks cluster tab, Update the values in below command:

$aws cloudformation create-stack --stack-name EKSworkerNodesCluster --template-body file://workerNodeGroup.yml --parameters   ParameterKey=ClusterName,ParameterValue=EKScluster  ParameterKey=ClusterControlPlaneSecurityGroup,ParameterValue=sg-0f6f7531b9a536493   ParameterKey=KeyName,ParameterValue=eks_key  ParameterKey=NodeGroupName,ParameterValue=EKSworkerNode  ParameterKey=Subnets,ParameterValue="subnet-03d23eb541c533e36\,subnet-04ae2a52ea0b67ffa\,subnet-061dbcdd424dcd20a" ParameterKey=VpcId,ParameterValue=vpc-078c1ab874fa1442a --capabilities CAPABILITY_IAM


###############Node discovery ###########
Copy ARN of instance role from Output section of above CFT statck

update the "rolearn" value in aws-auth-cm.yaml file and run below command.

Frirst time:
$ kubectl create -f aws-auth-cm.yaml 
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
configmap/aws-auth configured 

From second time onwards:
$ kubectl apply -f aws-auth-cm.yaml 


##Worker Node Stack with more options##
$aws cloudformation create-stack --stack-name nodeCluster --template-body file://node.yml --parameters   ParameterKey=ClusterName,ParameterValue=EKScluster  ParameterKey=ClusterControlPlaneSecurityGroup,ParameterValue=sg-07ee71ebad4e3db21  ParameterKey=KeyName,ParameterValue=eks_key ParameterKey=NodeAutoScalingGroupDesiredCapacity,ParameterValue=1 ParameterKey=NodeAutoScalingGroupMaxSize,ParameterValue=1 ParameterKey=NodeAutoScalingGroupMinSize,ParameterValue=1 ParameterKey=NodeGroupName,ParameterValue=NodeGroup ParameterKey=NodeInstanceType,ParameterValue=t2.micro ParameterKey=Subnets,ParameterValue="subnet-08c41497171e92daf\,subnet-00d7751b4e72bb55c\,subnet-03e2725fccdad5bb2" ParameterKey=VpcId,ParameterValue=vpc-0a196673b1630615a --capabilities CAPABILITY_IAM
