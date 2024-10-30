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
    
**Environment Variables**
•	registryCredential: AWS ECR credentials.
•	appRegistry: The ECR repository URL.
•	vprofileRegistry: The ECR registry URL.
•	cluster: The name of the ECS cluster.
•	service: The name of the ECS service.

**Pipeline Stages**

1. **Fetch Code**
stage('Fetch code'){
    steps {
        git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
    }
}

Description:
This stage retrieves the latest code from the docker branch of the VProfile GitHub repository. The code is fetched using the git command.

2. **Test Code**
   stage('Test'){
    steps {
        sh 'mvn test'
    }
}

Description:
In this stage, automated tests are executed using Maven. The mvn test command runs all the unit tests defined in the project to ensure code quality and functionality.

3. **SONAR QUBE Analysis**
   stage('build && SonarQube analysis') {
    environment {
        scannerHome = tool 'sonar4.7'
    }
    steps {
        withSonarQubeEnv('sonar') {
            sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                -Dsonar.projectName=vprofile-repo \
                -Dsonar.projectVersion=1.0 \
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        }
    }
}

Description:
This stage compiles the code and performs static analysis using SonarQube. The analysis checks for code quality, potential bugs, and vulnerabilities. It uses several parameters, such as project key, version, and paths for reports.

4. **Quality Gate**
   stage("Quality Gate") {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}

Description:
This stage Passes the Code through the define Quality Gate analysis. It will wait for an hour for the quality Gate to finish post whih it will abort the pipeline.

5. **Build App Image**
   stage('Build App Image') {
    steps {
        script {
            dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
        }
    }
}

Description:
This stage Builds the Docker image of the Code using Docker Build Command. It utilizes **appRegistry** variable which we have defined in the variables section.
Also it utilizes the $BUILD_NUMBER env variable of JENKINS to maintain different version of the code.

6. **Push the Image to ECR**
   stage('Upload App Image') {
    steps {
        script {
            docker.withRegistry(vprofileRegistry, registryCredential) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
            }
        }
    }
}

Description:
This stage uploads the Docker image to the specified Docker registry. Both the versioned image (tagged with the build number) and the latest image are pushed to the registry for deployment.

7. **Deploy to ECS**

   stage('Deploy to ECS') {
    steps {
        withAWS(credentials: 'awscreds', region: 'ap-south-1') {
            sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
    }
}

Description:
In the final stage, the application is deployed to Amazon ECS (Elastic Container Service). The AWS CLI command updates the ECS service to force a new deployment, ensuring that the latest version of the application is running.

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

