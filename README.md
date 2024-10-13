# CICD1
CICD Pipeline to fetch Code from Github ,make tests and quality analysis, create a docker image and push it to ECR then deploy it to ECS

Jenkins Pipeline for Dockerized Application Deployment
This repository contains a Jenkins pipeline configuration that automates the process of building, testing, and deploying a Dockerized application to AWS ECS. The pipeline integrates GitHub for source control, Maven for build and test management, SonarQube for code quality checks, and AWS ECR for Docker image storage.
Table of Contents
•	Features
•	Prerequisites
•	Pipeline Overview
•	Jenkinsfile Explanation
•	Setup Instructions
•	License
Features
•	Fetches code from GitHub
•	Runs Maven tests
•	Performs code analysis using Checkstyle
•	Builds a Docker image
•	Pushes the image to AWS ECR
•	Deploys the application to AWS ECS
Prerequisites
Before you can use this pipeline, ensure you have the following set up:
•	Jenkins: A Jenkins instance running with the necessary plugins installed (e.g., Docker Pipeline, AWS Sdk, SonarQube Scanner,Git, Pipeline AWS Steps, Maven).
•	AWS Account: Access to an AWS account with the availability of an IAM user with appropriate permissions to use ECR and ECS.
•	Access Key and Secret access Key for the above IAM Role
•	GitHub Repository: The source code should be hosted on GitHub.
•	SonarQube Sever URL and Credentials should be configured in the Manage Jenkins Section
•	SonarQube: Setup for code quality analysis.

Pipeline Overview
The pipeline consists of several stages:
1.	Fetch Code: Clones the specified branch from the GitHub repository.
2.	Test: Runs unit tests using Maven.
3.	Code Analysis with Checkstyle: Performs static code analysis.
4.	Performs Sonarqube Analysis
5.	Build App Image: Creates a Docker image from the code.
6.	Upload App Image: Pushes the built Docker image to AWS ECR.
7.	Deploy to ECS: Updates the ECS service to use the new image.
Jenkinsfile Explanation
Below is a breakdown of the key sections of the Jenkinsfile:
pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK17"
    }
    environment {
        registryCredential = 'ecr:ap-south-1:awscreds'
        appRegistry = "381492233214.dkr.ecr.ap-south-1.amazonaws.com/vprofilenew"
        vprofileRegistry = "https://381492233214.dkr.ecr.ap-south-1.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileappsvc"
    }
    




Environment Variables
•	registryCredential: AWS ECR credentials.
•	appRegistry: The ECR repository URL.
•	vprofileRegistry: The ECR registry URL.
•	cluster: The name of the ECS cluster.
•	service: The name of the ECS service.
Stages
Each stage is responsible for a specific part of the pipeline process, from fetching the code to deploying it on AWS ECS.
Setup Instructions
1.	Clone the Repository: Clone this repository to your local machine.
2.	Configure Jenkins:
o	Install necessary plugins (Docker Pipeline, AWS Sdk, Git, Pipeline AWS Steps, Maven).
o	Configure the AWS IAM credentials in Jenkins.
o	Set up Maven and JDK tools.
3.	Create a Jenkins Job:
o	Create a new pipeline job and link it to your GitHub repository.
o	Add the Jenkinsfile to your job configuration.
4.	Run the Pipeline: Trigger the pipeline to start the build and deployment process.

