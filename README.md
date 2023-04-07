# Final Project Report - Team Thunder Buddies

## Deploying Kubernetes Application using Amazon EKS

### Abstract
In this project, a containerized Photo Gallery application developed using WordPress is deployed onto a Kubernetes Cluster using spot instances. Amazon Elastic Container Service for Kubernetes (Amazon EKS) manages the clusters. EKS allows flexible running and deployment of Kubernetes applications in AWS cloud with all-time accessible and reliable clusters while automating features like patching and node provisioning. A single EKS cluster can run multiple applications by taking advantage of Kubernetes namespaces and IAM security policies. Spot Instances utilize unused EC2 instances at steep discounts lowering Amazon EC2 costs significantly. This makes the implementation a cost-efficient approach.

### Introduction
Kubernetes is open source that can be used within an on-premises infrastructure. It is cloud-agnostic, and cloud vendors provide Kubernetes as Platform as a Service (PaaS) to deploys and scale containerized applications across virtual machines. Amazon EKS is a fully managed Kubernetes service that makes it easy to deploy, manage, and scale containerized applications using Kubernetes on AWS. It facilitates easy migration any standard Kubernetes application to EKS. In this project, a WordPress application is likely to be deployed onto the Kubernetes cluster. 

### Improvements proposed
The proposed implementation suggests deploying the application using CloudFormation into the EKS Cluster. The photo gallery could be developed using WordPress which can be deployed using Helm charts with a prerequisite cluster formation deployed using the AWS quick start guide as reference. The improved implementation deploys the application into a cluster using spot managed node groups that also reduces the cost significantly up to 90% compared to deploying into a cluster with on-demand node groups using instance types such as m5.xlarge m5.large, m4.xlarge or m4.large which costs much higher. This also reflects the aim of our project in executing a Kubernetes application for less.

### Description
A.	Amazon EKS

Amazon EKS is a managed service used to Kubernetes across multiple AWS Availability Zones to ensure high availability, without needing to install, operate, and maintain an own Kubernetes control plane or nodes. It provides scalability and security to the applications including the following capabilities:
•	Amazon ECR for container images
•	Elastic Load Balancing for load distribution
•	IAM for authentication
•	Amazon  Virtual Private Cloud (VPC) for isolation
	Amazon EKS runs a single tenant Kubernetes control plane for each cluster which cannot be shared across clusters or AWS accounts. The cluster plane is actively monitored and auto scaled by EKS to ensure high performance. The Amazon VPC network policies controls network traffic components within the cluster. The control plane components for a cluster cannot view or communicate with other clusters or other AWS accounts, making it reliable for production workloads. The project starts with setting up Administrator access Permissions on the Cloud9 service.

B.	Kubernetes Resources

Kubernetes is an open-source software that allows deploying and managing containerized applications at scale. Kubernetes manages a cluster of compute instances and scheduling containers to run on the cluster with the available compute resources. Containers are run in logical groupings called pods and one or many containers can be run and scaled together as a pod. As Kubernetes is an open-source project, it can be used to run any containerized applications anywhere without needing to change the operational tooling. AWS works with the Kubernetes community, and contributes to the Kubernetes code base, to help Kubernetes users take advantage of AWS services and features. Kubernetes uses Helm a package manager to install, update third party applications called charts. These charts provide all the functionality needed by the Applications for running on the Kubernetes Cluster. The WordPress chart used in this project was once part of the Helm managed charts but recently it was taken over by VMware run package manager called Bitnami. Bitnami now provides the WordPress chart installation as a paid service compared to the WordPress provided by Helm which is free to install.

C.	The EKS Virtual Private Cloud (VPC)

The AWS VPC is an isolated virtual network with smaller network components such as subnets and internet gateways. A VPC is required to create a cluster and allows subnets at a minimum of two availability zones which can be configured as public or private. It is a common practice to configure a combination of public and private subnets to ensure the incoming traffic is load balanced. An Auto Scaling Group maybe used within every availability zone to monitor nodes withing any cluster, scaling them to fit the application's demands or to restart them on repetitive health probes failures.

D. Amazon EKS Cluster

The Amazon EKS Cluster consists of the Amazon EKS control plane and the nodes that are registered with it. Each Amazon EKS cluster control plane is single-tenant and unique that runs on its own Amazon EC2 instances. The EKS nodes run in the AWS account and use the API server endpoint to connect with the cluster’s control plane and a certificate file generated for the created cluster. Amazon EKS cluster is created using the eksctl command line tool. It creates an EKS control plane, and one node group. The Kubernetes Cluster-Autoscaler allows automatic adjusting of the number of nodes in the cluster as pods fail and are then rescheduled onto other nodes in the cluster. Spot managed node groups are added to the cluster which hep to increase the spot capacity in case the EC2 needs the capacity back. The instances are launched using Spot pools optimally chosen based on the available Spot capacity.

## System Setup:
### 1.	Setup Cloud9 environment.
AWS Cloud9 is a cloud-based integrated development environment (IDE) to write, run, and debug code within the browser with features of any IDE along with Linux server access and storage for code supporting over 40 programming languages. An IAM role with Administrator Access is required for the project setup. A “ccproject-role” is created using the IAM console with “AdministratorAccess” policy attached. This role will be used to manage the EC2 instance that is created by the AWS cloud9 environment. A new Cloud9 environment “ccproject-env” is created with AWS Linux OS from the AWS Console. This will initiate a stack in the CloudFormation and once the stack is fully created, the environment is available for access. We used a t2.micro EC2 instance as it is included in the AWS free tier account. On opening the environment from the Cloud9 environments tab, the manage EC2 option will direct to the EC2 instance used to host the cloud9 environment. The drop-down menu in the Actions option on the instance is used to modify the IAM role. The IAM role is changed to the “ccproject-role” created earlier. The AWS managed temporary credentials are turned off for the cloud9 environment created from the environment settings. Cloud9 manages IAM credentials temporary credentials that are incompatible with EKS IAM authentication. So, the credentials are disabled to connect with the IAM role created.


### 2.	Install command line tools.
The project requires setting up command line tools (CLI) for execution in the next steps. The three CLIs are installed to the “ccproject-env” using the Cloud9 terminal with the commands discussed below. The commands for installing AWS CLI are referred from the AWS CLI installation guide. The following commands install version 2 of AWS CLI in the cloud9 environment. Similarly, the Kubernetes command line tool (kubectl) is installed from the terminal using the commands referred from the Kubernetes docs. The following commands install the latest version of kubectl in the cloud environment. The Elastic Kubernetes Service command line tool (eksctl) is also installed from the terminal using commands taken from GitHub link sponsored by Weaveworks. Weaveworks eksctl is written in Go.

### 3.	Activate AWSQS third-party resources.
From the CloudFormation console under the Registry from the left panel the third-party AWS quick start resources required for this project are activated. The resources needed for this project are given below. 
AWSQS::EKS::Cluster
AWSQS::Kubernetes::Get
AWSQS::Kubernetes::Helm
AWSQS::Kubernetes::Resource

Helm is the package manager for Kubernetes to downloads charts which are pre-packaged, and pre-configured collections of the resources required for deploying a container. These charts are written in YAML and can be deployed to a Kubernetes cluster. These resources are published recently by the AWS Quick Start guide as a prerequisite to the deployment steps followed later in this project. Previously, the installation of Helm charts in Kubernetes was done by creating custom resources as per the requirement. Using the AWSQS::Kubernetes::Helm resource, we can install the necessary charts. Installing the charts using these third-party resources is like installing charts using command line tools. 

### 4.	AWS EKS Cluster creation using CloudFormation template.
The AWS EKS cluster for this project is created using a CloudFormation template. The template deployed in this project is referred using an AWS Quick Start that gives instructions to deploy an EKS Cluster. The AWS Quick Starts are referred as automated reference architectures that simplify the development of key workloads on the cloud. The Quick Start used deploys an EKS cluster into a new Virtual Private Cloud (VPC). The following is set up after successful deployment:
•	A highly available architecture that spans into several Availability Zones. Two availability zones are used: us-east-2a, us-east-2b.
•	A VPC configured with public and private subnets for a virtual network on AWS.
•	Managed NAT gateways in the public subnets that allow access for resources in the private subnets. 
•	An Amazon EKS cluster and Kubernetes nodes in the private subnets 

To deploy the Quick Start, the cluster information is entered such as the cluster name, stack name, and other parameters as required. It is important to set the Per-account shared resources and Per-Region shared resources to “Yes” to avoid any faulty detection by the AWS Auto detect option. On successful deployment, the cluster “ccprojcluster” is created using the stack “Amazon-EKS”.

### 5.	Add Spot Managed node groups to the EKS cluster
The Spot Managed node groups created in this project create and manage EC2 instances. Creating multiple spot capacities help increase the chances of maintaining capacity in case of interruptions. Two node groups are created with each node associated with several instance types. The spot pools are chosen optimally based on the available capacity to launch instances associated with each node group.

As part of the cluster deployed earlier, a ccprojcluster cluster is created. The node groups created in this step can be found in the compute tab for the ccprojcluster cluster’s configuration tab in the EKS service from the AWS management console.

EC2 Compute instances require managing storage which is achieved using Persistent Volume (PV), a resource object available in a Kubernetes cluster to define storage classes or storage implementations. A persistent volume claim (PVC) is a storage request like a pod that mounts a PV into a pod allowing it to consume PV resources.  The PVCs can request specific access modes and storage capacities to be mounted. Some available access modes are ReadOnlyMany, ReadWriteOnce, or ReadWriteMany. The PV storage is created using a YAML configuration file which specifies necessary specifications like access mode, storage capacity, storage classes, and any additional PVC policies required. 
### 6.	Deploy WordPress application
A photo gallery application is developed using WordPress. The application allows users to view favorite images and comment in the webpage. The photo gallery application can be accessed using the following URL: https://wordpress.com/view/srazzgallery.wordpress.com. The photo gallery application looks like the image below:

<img width="453" alt="image" src="https://user-images.githubusercontent.com/90299324/207703136-82828ad1-0b2b-419c-8289-9ae4758e3b0e.png">

This application is deployed to the EKS cluster using AWS CloudFormation. The CloudFormation template used installs WordPress Helm chart. The following commands to install the WordPress service on to a load balancer from command line using helm.


The CloudFormation template creates a stack named “wp-final ” using the parameters provided such as the cluster name, WordPress account credentials during stack creation. On successful deployment, the load balancer URL generated on the outputs tab of the newly created stack is used to log into the WordPress site. This shows successful deployment of a Kubernetes WordPress application into an Amazon EKS cluster.

### 7.	Challenges and Solutions
When creating the cluster in Step 4, third party cloud formation resources that are activated in step 3 must be deactivated. The Cloud formation stack automatically registers the third-party resources necessary for its creation privately. Once the cluster is created and you are ready to install the WordPress chart using the WordPress CloudFormation template stack provided in the GitHub. The second challenge is the requirement of a Lambda function which is needed to expose the WordPress hostname. This Lambda function is deployed when you create the cluster using AWS EKS Quick Start guide. This Deployment of cluster using AWS EKS Quick Start guide can be done in two ways. One method is to deploy the cluster into an existing VPC, and the second method is to deploy the cluster into a new VPC. In both the methods there is a need to install the per account resources and per region shared resources. 
These resources can be installed by choosing the AutoDetect feature provided by the AWS EKS Quick Start guide, but this feature fails to detect whether the resources are installed or not. Due to which the field must be set to Yes instead of AutoDetect. 

### 8.	Conclusion 
In this project, a Containerized Kubernetes Application developed using WordPress was deployed into AWS EKS service in a cost-efficient approach. This is achieved by running the node groups on spot instances which save up to 90% when compared to the same resource running on an On-Demand cluster node groups. Two different spot capacity pools (2vcpu-8gb and 4vcpu-16gb) are deployed in this project which helped to prevent the interruptions caused by over-utilization by maximizing the EC2 capacity. To provide a rough estimate, the entire project costed $5/day which approximates to $0.5/hour when the instances are up and running. 

### 9.	References
[1] V. C. Nwani, “Kubernetes to the cloud with AWS: Deploying a node.js app to Eks,” Okta Developer, 02-Nov-2021. [Online]. Available: https://developer.okta.com/blog/2021/11/02/k8s-to-the-cloud-aws [Accessed: 10-Oct-2022]. 

[2] M. Conn, “Run Kubernetes clusters for less with Amazon Elastic Kubernetes Service and Spot Instances.” Amazon, 2021. [Online]. Available: https://aws.amazon.com/getting-started/hands-on/amazon-eks-with- spot-instances/  [Accessed: 10-Oct-2022]. 

[3] B. Langston, “Eksworkshop.com,” Introduction :: Amazon EKS Workshop. [Online]. Available: https://www.eksworkshop.com/010_introduction/  [Accessed: 10-Oct- 2022]. 

[4] McConnell, J., Ameigh, T., & Gargan, A. (n.d.). Amazon EKS on the AWS Cloud Quick Start Reference Deployment. Amazon EKS on the AWS cloud. Retrieved November 14, 2022, from https://aws-quickstart.github.io/quickstart-amazon-eks/ 

[5] Amazon Web Services. (1991). Modular and Scalable Amazon EKS Architecture on AWS. Amazon. Retrieved November 14, 2022, from https://aws.amazon.com/solutions/implementations/amazon-eks/ 

[6] Ameigh, T. (2008). Using AWS CloudFormation to deploy software into Amazon EKS clusters. Amazon. Retrieved November 20, 2022, from https://aws.amazon.com/blogs/infrastructure-and-automation/using-aws-cloudformation-to-deploy-software-into-amazon-eks-clusters/ 
