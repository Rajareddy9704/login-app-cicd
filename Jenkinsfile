pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "rajareddy2000/terraform-login-app:latest"

        DOCKER_CREDENTIALS = "dockerhub-creds"

        DEPLOY_HOST = "98.91.206.134"

        SSH_CREDENTIALS = "ec2-ssh"

        APPROVER_EMAIL = "rajashekarreddybhumireddy@gmail.com"
    }

    stages {

        stage('Checkout') {

            steps {

                git(
                    branch: 'main',
                    url: 'https://github.com/Rajareddy9704/login-app-cicd.git'
                )
            }
        }


        stage('Docker Build') {

            steps {

                sh '''
                    docker build -t $DOCKER_IMAGE .
                '''
            }
        }


        stage('Docker Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push $DOCKER_IMAGE

                        docker logout
                    '''
                }
            }
        }


        stage('EC2 Deployment Approval') {

            steps {

                script {

                    emailext(
                        subject: "EC2 Deployment Approval Required - ${env.JOB_NAME} #${env.BUILD_NUMBER}",

                        to: "${APPROVER_EMAIL}",

                        body: """
Hello,

Docker image has been successfully pushed to Docker Hub.

Now EC2 deployment requires manual approval.

--------------------------------------------------

Application:
terraform-login-app

Docker Image:
${DOCKER_IMAGE}

Target EC2:
${DEPLOY_HOST}

Jenkins Build:
${env.BUILD_URL}

--------------------------------------------------

APPROVAL REQUIRED

Please open the Jenkins build using the link below:

${env.BUILD_URL}

The Jenkins pipeline is currently waiting for:

"Approve deployment to EC2?"

Click:

APPROVE

to deploy the application to EC2.

Click:

ABORT

to stop the deployment.

--------------------------------------------------

Deployment will happen ONLY after approval.

Regards,
Jenkins CI/CD
"""
                    )
                }


                timeout(time: 10, unit: 'MINUTES') {

                    input(
                        message: 'Approve deployment to EC2?',
                        ok: 'APPROVE'
                    )
                }
            }
        }


        stage('Deploy to EC2') {

            steps {

                sshagent(credentials: ["${SSH_CREDENTIALS}"]) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            ec2-user@$DEPLOY_HOST \
                            "docker pull $DOCKER_IMAGE && \
                             docker stop terraform-login-app || true; \
                             docker rm terraform-login-app || true; \
                             docker run -d \
                             --name terraform-login-app \
                             -p 80:80 \
                             $DOCKER_IMAGE"
                    '''
                }
            }
        }
    }


    post {

        success {

            emailext(
                subject: "EC2 Deployment Successful - ${env.JOB_NAME} #${env.BUILD_NUMBER}",

                to: "${APPROVER_EMAIL}",

                body: """
EC2 deployment completed successfully.

Application:
terraform-login-app

Docker Image:
${DOCKER_IMAGE}

EC2:
${DEPLOY_HOST}

Application URL:

http://${DEPLOY_HOST}

Jenkins Build:

${env.BUILD_URL}
"""
            )
        }


        failure {

            emailext(
                subject: "EC2 Deployment Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",

                to: "${APPROVER_EMAIL}",

                body: """
EC2 deployment failed.

Job:
${env.JOB_NAME}

Build:
${env.BUILD_NUMBER}

Jenkins Build:

${env.BUILD_URL}

Please check the Jenkins console output.
"""
            )
        }
    }
}
