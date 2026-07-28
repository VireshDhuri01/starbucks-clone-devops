pipeline {
    agent any
    tools {
        jdk 'jdk'
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION = "us-east-1"
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')])
                {
                git branch: 'main', url: 'https://github.com/VireshDhuri01/starbucks-clone-devops.git'
                sh "git pull --rebase https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/VireshDhuri01/starbucks-clone-devops.git main"
                }
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks \
                    -Dsonar.projectKey=starbucks'''
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy Code Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Image Build"){
            steps{    
                    sh "docker build -t ${params.ECR_REPO_NAME} ."
                }
            }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image ${params.ECR_REPO_NAME} > trivyimage.txt" 
            }
        }
        stage('Create ECR repo') {
            steps {
                withCredentials([string(credentialsId: 'access-key', variable: 'AWS_ACCESS_KEY'),
                                 string(credentialsId: 'secret-key', variable: 'AWS_SECRET_KEY')])
                    { sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY
                    aws configure set aws_secret_access_key $AWS_SECRET_KEY
                    aws ecr describe-repositories --repository-names ${params.ECR_REPO_NAME} --region ${AWS_REGION} || \
                    aws ecr create-repository --repository-name ${params.ECR_REPO_NAME} --region ${AWS_REGION}
                    """
                }
            }
        }
        stage('Login to ECR, Tag and Push Image') {
            steps {
                withCredentials([string(credentialsId: 'access-key', variable: 'AWS_ACCESS_KEY'),
                                 string(credentialsId: 'secret-key', variable: 'AWS_SECRET_KEY')])
                    { sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${params.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker tag ${params.ECR_REPO_NAME} ${params.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${params.ECR_REPO_NAME}:${BUILD_NUMBER}               
                    docker push ${params.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${params.ECR_REPO_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }
        stage('Cleanup Images') {
            steps {
                sh """
                docker rmi ${params.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${params.ECR_REPO_NAME}:${BUILD_NUMBER}
		        docker images
                """
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                script {
            def IMAGE = "${params.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${params.ECR_REPO_NAME}:${BUILD_NUMBER}"
            sh """ sed -i 's|image: .*|image: ${IMAGE}|' kubernetes/deployment.yml"""
                }
            }
        }
        stage('Push Manifest to GitHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) 
                    {sh """
                git config user.name "Viresh Dhuri"
                git config user.email "vireshdhuri28@gmail.com"
                git add kubernetes/deployment.yml
                git commit -m "Update image to build ${BUILD_NUMBER}" || true
                git push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/VireshDhuri01/starbucks-clone-devops.git HEAD:main
            """}
            }
        }
    }
}
