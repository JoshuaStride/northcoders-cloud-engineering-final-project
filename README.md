# Northcoders - Cloud Engineering - Final Group Project

### Designed by Joshua Stride, Joseph Adams, & Laura Woollard

This repository demonstrates the final group project in fullfillment of [Northcoders'](https://northcoders.com/our-courses/coding-bootcamp) [Cloud Engineering](https://northcoders.com/our-courses/cloud-engineering-bootcamp) bootcamp; a full-stack web application deployed using the software development and cloud architecture knowledge we have developed over the intensive, immersive thirteen week course, including:
- JavaScript, HTML5, CSS3, React, & Vite 💻
- Java, Maven, Springboot, Spring Actuator, & Spring JDBC 💽
- Amazon Web Services: 🌴
  - VPCs 🌤
  - S3 🪣
  - DynamoDB ⚡️
  - RDS 🌈
  - EC2 🧑🏻‍💻
  - IAM 🔐
  - EKS ⛴
  - SES 📧
  - SQS 📨
  - Lambda ƛ
- Infrastructure as Code (IaC) tooling: 👩🏾‍💻
  - Terraform 🌎
- Containerisation with Docker 🚚
- Kubernetes 🚢
  - Helm Charts ⚓️
- CI/CD ∞
  - CircleCI ⭕️
  - ArgoCD 🐙
- Telemetry & Observability 🔭
  - Prometheus 🔥
  - Grafana 📊

## Getting started...

The following setup guide allows you to recreate the cloud environment we provisioned using the resources in this repository. It assumes you have access to an AWS account, as well as local installation  of: ArgoCD, CircleCI, Docker, Helm, Kubernetes & kubectl, and Terraform.

You need to configure your AWS account to work with the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) and [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) using the documentation linked here. Once you are logged in to your AWS account via the terminal with your AWS credentials, we can proceed.

## Remote State Backend

First and foremost, we need to setup a secure **remote state backend** to improve security, reliability, and to facilitate teamwork in the development and operation of the software. A neater approach would be to provision an S3 bucket and DynamoDB table externally to this project, however for the purposes of knowledge exchange, we have included the neccesary configuration in this repository.

1) Firstly you need to specify the name we want to use for our remote state S3 bucket by specifying it as a variable value in the `.tfvars` file, and hardcoding the same value into line 6 of the `remote-state/backend.tf` file.
2) We can now run the config. In the `remote-state` directory: `providers.tf` and `backend.tf` **having commented out the terraform block therein**.
3) Once the S3 bucket and DynamoDB table have been provisioned, **uncommment** the terraform block and run `terraform init` to intialize the remote state backend.

## Virtual Private Cloud

Immediately we haved started provisioning infrastructure using Terraform. This means that we are already adherant to _The Golden Rule of Terraform_: once you've used Terraform, **only** use Terraform! Now, we are going to provision a Virtual Private Cloud (VPC) onto which we will deploy our application.

We can now navigate to the `infrastructure` directory and run `terraform apply` to provision our VPC using the IaC we have written. The VPC is a designed for failure, and is split across three availability zones with both public and private subnets for optimal security. It would not be capable of handling a production load, at this point, but does demonstrate the architectural principles we have employed. It is also custmoisable through parametrisation and the `terraform.tfvars` file.

## Elastic Kubernetes Service

By running `terraform apply` in the previous step, this will have created an empty Elastic Kuberneters Service cluster where our frontend and backend services will be deployed, using the IaC in the `containerisation` module.

EKS is a managaed Kubernetes service that allows you to run Kubernetes through AWS. The frontend and backend will later be deployed as nodes, where they can be accessed through the internet and communicate with one another to connect the different components of our application. In this development environment, only one replica of each has been deployed to keep costs low. In production, more replicas would be deployed to share the load of increased traffic, to keep availabilty and reliability high. EKS will also automatically run health check on nodes and, if unhealthy, restart them and reroute traffic away from that node.

## Elastic Container Registry

Next we will create the Elastic Container Registry repositories. This is where the docker images for the frontend and backend will be stored.

To complete the next step, you will need to navigate to the ECR modules: `ecr-terraform-be` & `ecr-terraform-frontend`. You then need to run `terraform init` and then `terraform apply` in each of the folders. Unfortunately, we weren't able to include this IaC in our main `infrastructure` module, as the ECR repositories need to be provisioned in a different AWS location than the context of our deployment. We would prioritise attending to this if we were continuing further with the project.

## Relational Database Service

The deployed application uses a Amazon Relational Database Service (RDS) database to store user information, allowing users to create and log into their accounts from different machines. For this project, the database has already been set up and the EKS deployment accesses it by using the credentials including username, password, name, endpoint of the database and the port through which to access the database. The database was configured with public access and an ingress security group allowing access through the corresponding postgreSQl port 5432. To keep costs low, a smaller database has been used in the devlopment stage. When going to production, it is likely a bigger database will need to be configured. 

## Helm

The original Kubernetes deployment and service charts that we used to check we could deploy the applciation locally were refactored into Helm charts. The Helm charts did not make any changes to the deployment of the applications, but was a way to better organise the deployment and service files, and allowed for a more efficient way to update the configuration of the deployment.

The user may need to reconfigure these Helm charts. Specifically, in the  **values.yaml** for the frontend and backend, the **image repository** (line 8) may need to be changed to the **URI** of the corressponding frontend/backend ECR repositiroy. The URIs for the frontend and backend can be found by going into public reposisotires in Amazon Elastic Container Service.

## CircleCi

CircleCi is the platform used for the continuous integration stage of the CI/CD pipeline. When new code is pushed to the main branch of this repository, CircleCi will build the image of the frontend and backend and run each of the test scripts. If all the code compiles without errors and the tests pass, the updated images will be pushed up to their corresponding ECR repositories. This means that new code can be continuously integrated into the ECR image repository. 

Firstly, `config.yml` in `.circleci` must be set up: this is the configuration file that CircleCi will use to build images and run tests on them in the correct order. Look through this file if you wish to gain a better understanding. Some changes may be necessary for the `config.yml` to ensure it uses the correct ECR repository.

The **public-registry-alias** for the frontend and backend (lines 50 and lines 73 in `config.ymal`) will need to be changed to match your public registry alias. This is the set of letters and numbers found in the URI of your public repositories, between public.ecr.aws/ and the repository name; If your URI is **public.ecr.aws/a1b2c3d4/node-api-circleci**, your public-registary-alias will be **a1b2c3d4**.

Afterwards, you will need to set up the relevant permissions. To step this up, first head to CircleCi and sign in/up using the same GitHub account that has access to this repository. In **Projects**, there should be a list of your repositories. Select **Set Up Project**, select **Fastest** and point CircleCI to the main branch of this repository.

After this you will need to set up some credentials.
  1) Head to IAM in AWS.
  2) Under Users, select Add users. Give yourself user a username and select next.
  3) In Permission options, select Attach policies directly.
  4) In Permission policies, search for and select:
    - **AmazonElasticContainerRegistaryFullAccess**
    - **AmazonEC2ContainerRegistaryFullAccess**
  5) Select Next and Create User.
  6) Once it has been created, go to Security credentials in the new user.
  7) Create Access Key and select Third-party Service.
  8) Set a description and then Create Access Key. This will give you an Access key and Secret access key.
  9) Select Download .csv file so you can access them.

Once you have created the relevant AWS access credentials, they need to be applied to the CircleCi project.
  1) Go into Project Settings of your project and into environment variables.
  2) Select Add Environment Variable and add the following variables:
  ```
  AWS_ACCESS_KEY_ID - Value will be inside .csv file
  AWS_ECR_REGISTRY_ID - Value is your 12 digit AWS Account ID
  AWS_SECRET_ACCESS_KEY - Value will be inside .csv file
  ```
Whenever new code is pushed to GitHub, CircleCi will now automatically run tests on it and push the images up to the relevant ECR repository.

## Argo CD

Argo is a kubernetes controller which continuously monitors running applications and compares the current, live state against the desired target state (as specified in the Git repo).

To access ArgoCD, you will first need to make sure you are in the correct kubernetes context. To do this, run:

  `aws eks --region eu-west-2 update-kubeconfig \ --name tt-project-cluster`

We now need to set up the Argo namespace:

  `kubectl create namespace argocd` 

and then download and apply the argo yaml:

  `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` 

To check this has all gone through and you are set up correctly, you can use this command:

`kubectl get pods -n argocd`

To access Argo, you will need to port-forward and you will need two terminals for this. Using the port which is given to you next to argocd-server when you run `kubectl get pods -n argocd`.

Run `kubectl port-forward svc/argocd-server -n argocd 8080:443` and then log on to your browser and access localhost:8080. You should see the argo homepage.  In your other terminal, run this command which will give you the password needed to log-in on the browser with the username as **admin**. 

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

On the ArgoCD dashboard, you will need to navigate to the repo section and add this repo to argo. You will need to use a [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) to link your github repo to Argo securely. 

Once the repo has been connected, you will need to set up the backend application. Select **add app** and make sure that you name the app semantically, and select the project as *default*. In **source**, choose the repo url to be the one you've just set-up and select the **path** as the backend folder of the repo. In **destination**, select the cluster as the default link which appears and name the namespace to be **default**. 

Click **create**, then **refresh** and **sync** the app.

*Repeat the above for the frontend app!*

## Observability

Now we need to do something similarto deploy Prometheus with Argo. However, we will be using [this helm chart](https://artifacthub.io/packages/helm/prometheus-community/prometheus) instead of a GitHub repository. The set up of the app follows a similar process - **install the helm chart at the above link using Argo**.

Prometheus can be used for analysing metrics and we can set up custom metrics to look at specific aspects of our app once it has been provisioned. For example, we have set up Prometheus in such a way that if you access the app with the endpoint '../actuator/prometheus' you can see a list of metrics.

## Grafana

Although Prometheus is tremendously useful when it comes to moitoring application deployments, diagnosing issues, and preventing errors before they occur, the raw data it outputs is not the most effective way of using its metrics. We can use Grafana to visualise this data, and make it easier to find and fix issues when they occur. Below is one example of a data visualisation dashboard we set-up using Grafana:

![A screenshot of a Grafana dashboard, visualisting telemetric data related to the deployed applicaiton](https://github.com/JoshuaStride/northcoders-cloud-engineering-final-project/assets/127965188/62f42e18-b1a6-4121-8b5c-ace88bd1c860)


## Lambda

When provisioned in the AWS console for testing and learning purposes, we were ablke to create a Lambda function that automatically sent emails to the operations team, alterting them when a new user signs up for the application, as well as an email to the new user welcoming them. However, integrating the function into the Terraform IaC tooling is still an active ticket. When fully integrated, it will create the Lambda function through Terraform and assign to it the relevant roles and policies. The code for the Lambda function can be found in `index.mjs` in the **infrastucture** directory.

Furthermore, Amazon Simple Email Service (the service Lambda uses to send emails) is in Sandbox mode. This means we can only send emails to email addresses verified in Verified identities. To get into production mode where emails can be sent to unverified addresses (new users who have just created an acount in our application), a request needs to be sent to Amazon, where it will be reviewed.
