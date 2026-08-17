pipeline {
    agent { label "jenkins-agent" }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
    }
    environment {
        APP_NAME   = "register-app"
        AWS_REGION = "us-east-1"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/devaasirvathamsj/gitops-register-app.git'
            }
        }
        stage("Resolve ECR Registry") {
            steps {
                script {
                    env.AWS_ACCOUNT_ID = sh(
                        script: "aws sts get-caller-identity --query Account --output text",
                        returnStdout: true
                    ).trim()
                    env.IMAGE_NAME = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"
                }
            }
        }
        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's|image: .*|image: ${env.IMAGE_NAME}:${params.IMAGE_TAG}|g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }
        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "devaasirvathamsj"
                   git config --global user.email "devaasirvathamsj@gmail.com"
                   git add deployment.yaml
                   git diff --staged --quiet || git commit -m "Updated Deployment Manifest to tag: ${params.IMAGE_TAG}"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/devaasirvathamsj/gitops-register-app.git main"
                }
            }
        }
    }
}
