# Kubernetes Dummy Application Project Deployed on AWS EKS

This project serves as a demonstrative dummy application using Kubernetes, consisting of multiple microservices. The deployment instructions cover production on both Local and AWS EKS. Please note that the code for the app, including the JavaScript, HTML, JSON and CSS is not authored by the repository owner.

## - Production Deployment - Local System -

### Prerequisites

To run this project, ensure the following prerequisites are met:

- Docker installed on your machine
- Docker Hub account to store your Docker images
- kubectl installed locally to interact with Kubernetes clusters
- Minikube installed for local Kubernetes development
- MongoDB Atlas account for the database (free tier applicable)
- Postman app

### Getting Started

1. Clone the Repository:

    ```bash
    git clone https://github.com/Enoch-Gonzalez/07-kubernetes-multiples-microservices-eks-depl.git
    ```

    ```bash
    cd /07-kubernetes-multiples-microservices-eks-depl
    ```

### Setting Up the Project

1. Navigate in your MongoDB Atlas account and in your cluster do the following:
	
    a.	Click on connect
	b.	Click on Drivers
	c.	In the number 3 copy the url(string between @<copy-this-string>/)

3. Replace each deployment on the kubernetes directory in every tasks.yaml file and users.yaml file.

- Under the deployment resource section, locate the value variable under the container>env section. Replace the value:

    ```
    mongodb+srv://<user>:<password>@cluster0.m4xru7x.mongodb.net/?retryWrites=true&w=majority
    ```

- Replace the user and password.

4. Create a Docker Hub repository for your images.

5. Build Docker images for each service and tag them with your Docker Hub repository.

- Navigate into each service directory and create Docker images:

    + **Auth API**

        ```bash
        cd auth-api
        ```

       ```bash 
        docker build -t <DOCKERHUB_USERNAME>/auth-api .
        ```

        ```bash
        docker push <DOCKERHUB_USERNAME>/auth-api
        ```

    + **Tasks API**

        ```bash
        cd ../tasks-api
        ```
    
        ```bash
        docker build -t <DOCKERHUB_USERNAME>/tasks-api .
        ```

        ```bash
        docker push <DOCKERHUB_USERNAME>/tasks-api
        ```

    + **Users API**

        ```bash
        cd ../users-api
        ```

        ```bash
        docker build -t <DOCKERHUB_USERNAME>/users-api .
        ```

        ```bash
        docker push <DOCKERHUB_USERNAME>/users-api
        ```

    Push the images to your Docker Hub repository.

    These commands provide a clear pathway for navigating into each service directory, building Docker images, and pushing them to your Docker Hub repository. Adjust `<DOCKERHUB_USERNAME>` to your actual Docker Hub username.

6. Check Minikube Status:

- Before deploying, ensure Minikube is up and running:

    ```bash
    minikube status
    ```

- If Minikube is not running, start it using the configured driver:

    ```bash
    minikube start --driver=<your-drivert-set-when-installing-minikube-docker-or-virtualvm>
    ```

7. Deploy Kubernetes Objects:

- Before applying the YAML files, ensure to update them with your specific repository and image names. Also ensure that the replicas for each deployment is set to 1.

- Navigate to each `deployment.yaml` file in the `kubernetes` directory. Look for the `spec -> containers -> image` field in each file and replace `<DOCKERHUB_USERNAME>/<IMAGE_NAME>` placeholders with your Docker Hub username and chosen image names.

- Here's an example snippet of the YAML file where you should make the changes:

    ```yaml
    spec:
        containers:
            - name: auth
              image: <DOCKERHUB_USERNAME>/auth-api:latest # Replace this line with your Docker Hub username and image name
    ```

8. Apply the YAML files to create Kubernetes objects:

    ```bash
    kubectl apply -f kubernetes/auth-deployment.yaml
    kubectl apply -f kubernetes/auth-service.yaml
    ```

    ```bash
    kubectl apply -f kubernetes/tasks-deployment.yaml
    kubectl apply -f kubernetes/tasks-service.yaml
    ```

    ```bash
    kubectl apply -f kubernetes/users-deployment.yaml
    kubectl apply -f kubernetes/users-service.yaml  
    ```

9. Check Service Status: 

    ```bash
    kubectl get service users-service
    ```

- The external IP might show as "Pending" because Minikube doesn't provide an external IP for services of type LoadBalancer. For cloud providers like AWS, an external IP would be displayed here.

10. Access the Application in Minikube:

    ```bash
    minikube service users-serivce
    ```

11. Signup to the app:

- To sign up to the app send a post request to /signup using Postman:

    + Add a JSON raw with the following:

        ```json
        {
        “email”: ”test@test.com”
        “password”: ”password”
        }
        ```
    
    + Send a POST request to /signup:
        
        ```
        http://<minikube-serivce-users-service>/signup
        ```

12. Signin to the app:

- Once you receive a message that the user was successfully created sent a post request to /login using Postman:

    + Add a JSON raw with the following:

        ```json
        {
        “email”: ”test@test.com”
        “password”: ”password”
        }
        ```

    + Send a POST request to /login
        
        ```
        http://<minikube-serivce-users-service>/login
        ```

- This will throw a token. This will throw a token. Copy it.

13. Add tasks to your app.

- Add tasks by adding a Header in Postman. This require authentication, add a Header and send a post Request:

    ```
    Key: Authorization
    Value: Bearer <copy-token>
    ```

    + Add a JSON raw with the following:

        ```json
        {
        “Title”: ”Learn Kubernetes”
        “Text”: ”Learn it in-depth”
        }
        ```

    + Send a POST request to /tasks:
        
        ```
        http://<minikube-serivce-users-service>/tasks
        ```

- This task will be stored and since the project has a volume the task will persist.

14. Delete a task.

- Send a delete request with a Header with the user’s token who created the task:

    ```
    Key: Authorization
    Value: Bearer <copy-token>
    ```

    + Send a DELETE request to:
        
        ```
        http://<minikube-serivce-users-service>/tasks/<id-of-task>
        ```

### Cleaning Up

1. To delete the resources created by the YAML file, use the following command:

    ```bash
    kubectl delete -f kubernetes/auth-deployment.yaml
    kubectl delete -f kubernetes/auth-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/frontend-deployment.yaml   
    kubectl delete -f kubernetes/frontend-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/tasks-deployment.yaml
    kubectl delete -f kubernetes/tasks-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/users-deployment.yaml
    kubectl delete -f kubernetes/users-service.yaml
    ```

- This will remove the service, deployment, persistent volume claim, and the persistent volume from your Kubernetes cluster.

2. Stop Minikube:

    ```bash
    minikube stop
    ```

3. Delete Minikube:

    ```bash
    minikube delete
    ```

4. To remove docker Containers if docker driver was used to start Minikube:

    ```bash
    docker ps -a -q | xargs docker rm
    ```
    
5. To remove docker images if docker driver was used to start Minikube:

    ```bash
    docker images -q | xargs docker rmi
    ```

- The above commands will stop and delete Minikube and then remove all Docker containers and images on your system. Be cautious when using the docker rm and docker rmi commands, as they will remove all containers and images, not just those related to Minikube.

## - Production Deployment - AWS EKS -

### Prerequisites

- Docker installed on your machine
- Docker Hub account to store your Docker images
- kubectl installed locally to interact with Kubernetes clusters
- Minikube installed for local Kubernetes development
- AWS account with EKS access
- Postman app

### Configure the Dummy Application on AWS EKS using the mgmt console

1. Go to the EKS service.

2. Create and configure the EKS Cluster

- Cluster name: kub-dep-demo. Click Next step.
- kubernetes version: 1.23
- Closet Service Role: Click on IAM Console:
- In the saearch box select: EKS - Cluster, Next
- Filter policies, Next
- Omit tags, Next
- Role Name: eks-cluster
- Create role
- Select Service Role: eks-cluster
- Next

**Specify Networking**
- Search in All services for CloudFormation and open it in new tab
- Click Create Stack
- Amazon S3 URL: go to this url and copy the IPV4 url = <https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html#create-vpc>= paste the url, Next
- Leave all as default, Next
- Stack name: eksVpc, Next
- Omit and leave everything as default
- Create stack
- Back to  Specify Networking
- VPC: eksVpc
- Cluster endpoint access
- Select Public and private
- leave everything as default, Next

**Configure logging**
- Leave everything as default, Next
- Review and create, Click in create

Wait until the cluster is created.

3. Configure your CLI to have access to the EKS Cluster

**Configure the Minikube config file**
- Go to your local system user folder and look for the hidden file “.kube” directory
- Create a copy of the actual “config” file and rename it with an appropriate name for you to remember that this file holds the information for Minikube to your local machine
- Override the config file with the configuration for AWS.
- Go to: https://aws.amazon.com/cli/
- Download and install the installer for your local system
- Confirm all default settings of the installation
- Go to AWS: your-user-name>My security credentials> Access keys> Create a new Access key> Download Key File. Save this file
- Open the Access Key file with a text editor and leave it open to have access to the Access Key and the Secret Key
- Open a terminal and run

    ```bash
    aws configure
    ```
    
    ```
    >enter the AWS ACCESS KEY ID
    >enter the AWS SECRET KEY
    >enter REGION : region where you EKS Cluster is in
	```
    
- On your terminal run the following:

    ```bash
    aws eks --region <enter-your-aws-region> update-kubeconfig --name kub-dep-demo
    ```

- With this new configuration al the kubectl commands will run in inside your AWS EKS Cluster

4. Adding worker nodes

- Under your EKS Cluster dashboard go Compute> Add Node Group

**Configure Node Group**
- Name: demo-dep-nodes
- Node IAM Role - Click “IAM console”
- Create Role
- In the saearch box select: EC2, Next
- Filter policieS: select and add: eksworker + cni + ec2containerRegistryReadOnly
- Omit tags, Next
- Role Name: eksNodeGroup
- Create role
- Select Service Role: eksNodeGroup
- Next

**Set Compute and scaling configuration**
- Instance type: t3.micro
- Leave everything as default, Next

**Specify Networking***
- Disable Allow remote access to nodes

**Review**
- Click Create

5. Configuring EFS as a Volume

- Open a terminal to install the EFS CSI driver and copy the following url to copy the driver inside the cluster.

    ```bash
    kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.7"
    ```

- src: https://github.com/kubernetes-sigs/aws-efs-csi-driver

**Go to EC2 dashboard**
- Click on Security Group

**Create Security Group**
- security group name: eks-efs
- VPC: eksVpc
- Inbound rule: Add rule
- type: NFS
- Source: Custom
- CIDR blocks: <copy the IPv4 CIDR of your VPC created for your EKS>
- Leave everything as default
- Create security group

**EFS**
- Go to EFS  on AWS
- Click on Create file system
- name: eks-efs
- VPC: eksVpc
- Click customize
- Network access for both availability zones
- security group: eks-efs
- Click Create
- Copy and paste somewhere the EFS File System ID
- Go to the documentation, under the users-api.yaml file replace the “volumeHandle” value to the ID copied in the previous step

### Getting Started

1. Clone the Repository:

    ```bash
    git clone https://github.com/Enoch-Gonzalez/07-kubernetes-multiples-microservices-eks-depl.git
    ```

     ```bash
    cd /07-kubernetes-multiples-microservices-eks-depl
    ```

2. Create a Docker Hub repository for your images.

3. Build Docker images for each service and tag them with your Docker Hub repository.

- Navigate into each service directory and create Docker images:

    **Auth API**
    ```bash
    cd auth-api
    ```

    ```bash
    docker build -t <DOCKERHUB_USERNAME>/auth-api .
    ```

    ```bash
    docker push <DOCKERHUB_USERNAME>/auth-api
    ```

    **Tasks API**
    
    ```bash
    cd ../tasks-api
    ```

    ```bash
    docker build -t <DOCKERHUB_USERNAME>/tasks-api .
    ```
    
    ```bash
    docker push <DOCKERHUB_USERNAME>/tasks-api
    ```

    **Users API**
    
    ```bash
    cd ../users-api
    ```

    ```bash
    docker build -t <DOCKERHUB_USERNAME>/users-api .
    ```

    ```bash
    docker push <DOCKERHUB_USERNAME>/users-api
    ```

    Push the images to your Docker Hub repository.

    These commands provide a clear pathway for navigating into each service directory, building Docker images, and pushing them to your Docker Hub repository. Adjust `<DOCKERHUB_USERNAME>` to your actual Docker Hub username.

4. Deploy Kubernetes Objects:

Before applying the YAML files, ensure to update them with your specific repository and image names.

Navigate to each `deployment.yaml` file in the `kubernetes` directory. Look for the `spec -> containers -> image` field in each file and replace `<DOCKERHUB_USERNAME>/<IMAGE_NAME>` placeholders with your Docker Hub username and chosen image names.

Here's an example snippet of the YAML file where you should make the changes:

    ```yaml
    spec:
    containers:
        - name: auth
        image: <DOCKERHUB_USERNAME>/auth-api:latest # Replace this line with your Docker Hub username and image name
    ```


5. Apply the YAML files to create Kubernetes objects:

    ```bash
    kubectl apply -f kubernetes/auth-deployment.yaml
    kubectl apply -f kubernetes/auth-service.yaml
    ```

    ```bash
    kubectl apply -f kubernetes/tasks-deployment.yaml
    kubectl apply -f kubernetes/tasks-service.yaml
    ```

    ```bash
    kubectl apply -f kubernetes/users-deployment.yaml
    kubectl apply -f kubernetes/users-service.yaml
    ```

6. Check Service Status:

    ```bash
    kubectl get service users-service
    ```

**Copy the EXTERNAL-IP of the users-service**

7. Signup to the app:

- In order to access the app and to sign up send a post request to /signup using Postman using the load balancer ip copied:

    + Add a JSON raw with the following:

        ```json
        {
        “email”: ”test@test.com”
        “password”: ”password”
        }
        ```

    + Send a POST request to /signup:
        
        ```
        http://<EXTERNAL-IP-serivce-users-service>/signup
        ```  

8. Login to the app:

- Once you receive a message that the user was successfully created sent a post request to /login using Postman:


    + Add a JSON raw with the following:

        ```json
        {
        “email”: ”test@test.com”
        “password”: ”password”
        }
        ```

    + Send a POST request to /login:
        
        ```
        http://<EXTERNAL-IP-serivce-users-service>/login
        ```  

**This will throw a token. Copy it.**

9. Add tasks to your app.

- Add tasks by adding a Header in Postman. This require authentication, add a Header and send a post Request:

    ```
    Key: Authorization
    Value: Bearer <copy-token>
    ```

    + Add a JSON raw with the following:

        ```json
        {
        “Title”: ”Learn Kubernetes”
        “Text”: ”Learn it in-depth”
        }
        ```

    + Send a POST request to /tasks:

        ```
        http://<EXTERNAL-IP-serivce-users-service>/tasks
        ```

        This task will be stored and since the project has a volume the task will persist.

10. Delete a task

- Send a delete request with a Header with the user’s token who created the task:

    ```
    Key: Authorization
    Value: Bearer <copy-token>
    ```

    + Send a DELETE request to: 
    
        ```
        http://<EXTERNAL-IP-serivce-users-service>/tasks/<id-of-task>
        ```

### Cleaning Up 

1. Cleanup - AWS EKS

- To delete the resources created by the YAML file, use the following command:

    ```bash
    kubectl delete -f kubernetes/auth-deployment.yaml
    kubectl delete -f kubernetes/auth-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/frontend-deployment.yaml
    kubectl delete -f kubernetes/frontend-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/tasks-deployment.yaml
    kubectl delete -f kubernetes/tasks-service.yaml
    ```

    ```bash
    kubectl delete -f kubernetes/users-deployment.yaml
    kubectl delete -f kubernetes/users-service.yaml
    ```

2. Delete the AWS resources.

**Delete EFS File System**
- Go to AWS Management Console and navigate to Amazon EFS.
- Select the file system associated with your project (named 'eks-efs').
- Click on 'Actions' and choose 'Delete file system'.
- Confirm the deletion when prompted.

**Remove EFS Security Group**
- Access the AWS Management Console and go to the EC2 dashboard.
- Select 'Security Groups' from the menu.
- Find and select the security group named 'eks-efs'.
- Click 'Actions' and then 'Delete security group'.
- Confirm the deletion when prompted.
- Delete EKS Cluster and Associated Resources
- Open the AWS Management Console and navigate to the Amazon EKS dashboard.
- Select the EKS cluster created for your project (named 'kub-dep-demo').
- Click 'Delete cluster' and follow the prompts to confirm deletion.

**Delete IAM Roles**
- Navigate to the AWS IAM Console.
- Select 'Roles' from the menu.
- Look for the roles created during the EKS setup (roles like 'eks-cluster' and 'eksNodeGroup').
- Select each role and click 'Delete role'.
- Confirm the deletion when prompted.

**Remove CloudFormation Stack**
- Access the AWS Management Console and go to the CloudFormation dashboard.
- Locate and select the stack named 'eksVpc'.
- Click 'Delete' and follow the prompts to delete the stack.

**Clean Up EKS Node Group**
- Go to the Amazon EKS dashboard and select 'Node Groups'.
- Find the node group created during the setup (named 'demo-dep-nodes').
- Click 'Delete' and confirm the action.

Make sure to delete all the resources that were created in order to avoid high costs.

#### Disclaimer Regarding AWS Charges

Please note that deploying resources on AWS, including but not limited to Amazon EKS, EFS, EC2 instances, and associated services, may incur charges based on your usage and the AWS pricing model.

**User Responsibility:**
- Users are responsible for understanding and managing the costs associated with using AWS services, including the creation, deployment, and deletion of resources mentioned in this documentation.
- I strongly advise users to review the AWS pricing details and estimate the potential costs before initiating any actions mentioned in this guide.

    - Liability Waiver:
	    + The instructions provided in this documentation are for educational and demonstrative purposes only. While every effort has been made to ensure accuracy, I cannot guarantee that following these steps will prevent incurring AWS charges.
	    + The creator(s) of this project and associated documentation cannot be held responsible for any unexpected or unintended charges that may occur on your AWS account as a result of following these instructions.

    - Recommendations:
	    + Before executing any actions described in this guide, users should review and comprehend the AWS pricing structure, billing details, and any applicable free-tier options to avoid unexpected charges.
	    + It is advised to regularly monitor your AWS account for any unused or unnecessary resources and promptly delete them to prevent unnecessary charges.

By proceeding with the instructions mentioned in this documentation, you acknowledge and agree that you understand the potential financial implications and release the creator(s) of this project from any liability related to AWS charges incurred during or after following these instructions.

#### Note

This project demonstrates how to deploy a dummy project imitating both a development and production phase.

### License

This project is licensed under the MIT License - see the LICENSE file for details.