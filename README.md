# Terraform NCAA Game-Highlights Automation

This Project leverages Terraform as Infrastruce as a code (IaC) tool to automate sports Highlights with AWS MediaConvert. RapidAPI is used to obtain game highlights using Docker containers and AWS Elemental Converter to transcode game highlights into multiple resolutions for different platforms. 
## This is a follow up of [Part 1](https://github.com/olanuges90/Highlights-ProcessorNCAA) 

## Technical Diagram
![highlightsprocessor drawio](https://github.com/user-attachments/assets/fdf91dfc-fe8a-4efa-819a-70a2a512ab2f)


## File Overview
- config.py: Handles environment variables for dynamic and flexible configuration management for various environments.  (e.g development, staging and production).
- fetch.py: Fetches NCAA game highlights from RapidAPI and saves them as a JSON format in S3 Bucket as a JSON format(basketball_highlights.json).
- process_one_video.py: Downloads the first video from the JSON file , saves it in the S3 Bucket under the videos folder and logs each step.
- mediaconvert_process.py: Configures MediaConvert jobs for video processing and stores the output in S3.
- run_all.py: Executes all scripts sequentially incorpoerating buffer time between tasks.
- .env: Securely stores environment variables (i.e Variables we don't want to hardcode).
- Dockerfile: Specifies the instrucions for building the Docker image.

## Prerequisities
1. RapidAPI Account: Create an account for accessing NCAA highlights and Images.
   
   API Endpoint: [Sport Highlights API](https://rapidapi.com/highlightly-api-highlightly-api-default/api/sport-highlights-api/playground/apiendpoint_16dd5813-39c6-43f0-aebe-11f891fe5149)

2. Installations Required and verify.
    - Docker: Download and Install Docker desktop, verify with docker --version
    - AWS CLI: Install AWS CLI. Verify Installation: aws --version. Set up AWS Credentials with command: aws configure
    - Python 3: Install Python 3. Verify Installation: python --version
  
3. AWS Account ID: Retrieve from the AWS Management Console.
    - Go to your AWS account, click on your account name in the top right corner and copy your account ID. Keep in a safe place.
4. AWS Credentials: Retrieve ACCESS Key and Secret Key
   - Log in to the AWS Management Console.
   - Go to IAM (Identity and Access Management).
   - Navigate to Users and select your user.
   - Click Security credentials.
   - Under Access keys, click Create access key.
   - Copy and store the Access Key ID and Secret Access Key securely.
  
  ## Project Strcuture

     resources/
      ├── vpc_setup.sh
      ├── ncaaprojectcleanup.sh
      ├──IAMPolicies
     src/
      ├── Dockerfile
      ├── config.py
      ├── fetch.py
      ├── process_one_video.py
      ├── mediaconvert_process.py
      ├── run_all.py
      ├── requirements.txt
      ├── .env
      └── .gitignore
      └── terraform
          ├── main.tf
          ├── variables.tf
          ├── secrets.tf
          ├── iam.tf
          ├── ecr.tf
          ├── ecs.tf
          ├── s3.tf
          ├── container_definitions.tpl
          └── outputs.tf

  ## SetUp Instructions
  1. In the github repo, there is a resources folder. Copy the entire contents
  2. In the VScode terminal, create the file vpc_setup.sh and paste the script inside.
  3. Run the script

    bash vpc_setup.sh

4. You will see variables in the output, paste these variables into lines 8-13.
5. Store your API key in AWS Secrets Manager

``` sh
aws ssm put-parameter \
  --name "/myproject/rapidapi_key" \
  --value "YOUR_SECRET_KEY" \
  --type SecureString
   ```
6. Run the following script to obtain your mediaconvert_endpoint

``` sh
aws mediaconvert describe-endpoints --query "Endpoints[0].Url" --output text
```
7. Leave the mediaconvert_role_arn string empty

## Run The Project
1. Navigate to the terraform folder/workspace in VS Code From the src folder
```sh
cd terraform
```
2. Initialize terraform working directory
```sh
terraform init
```
3. Check syntax and validity of your Terraform configuration files
```sh
terraform validate
```
4. Display execution plan for the terraform configuration
```sh
terraform plan
```
5. Apply changes to the desired state
```sh
terraform apply -var-file="terraform.dev.tfvars"
```
6. Create an ECR Repo
```sh
aws ecr create-repository --repository-name highlight-pipeline
```
7. Log into ECR
```sh
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```
8. Build and Push the Docker Image
```sh
docker build -t highlight-pipeline:latest .
docker tag highlight-pipeline:latest <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/highlight-pipeline:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/highlight-pipeline:latest
```
9.  Run the Highlight Processing Container
```sh
docker run --env-file .env highlight-processor
```
10. Verify
In the AWS Management console,
1. Naviage to AWS Elemental MediaConvert, verify the Job is "Completed"
2. Navigate to S3 bucket to view the Newly created bucket and the files in it (Processed videos, First video, highlights)
   
## Bonus: Destroy ECS and ECR resources
1. In the AWS Cloudshell or vs code terminal, create the file ncaaprojectcleanup.sh and paste the script inside from the resources folder.
2. Run the script
```sh
bash ncaaprojectcleanup.sh
```
What we Learned: 
1. Deploying local docker images to ECR
2. Using Terraofm as IaC to automate the creation and deployment of Networking - VPCs, Internet Gateways, private subnets and public subnets
3. SSM for saving secrets and pulling into terraform
  
 
      
