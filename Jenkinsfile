pipeline {
    agent { label 'Jenkins-Slave-node-label' }

    tools {
        maven 'Maven'
    }

    environment {
        registry   = "accountid.dkr.ecr.ap-south-2.amazonaws.com/repo-name"
        imageTag   = "v1.${BUILD_ID}"
        fullImage  = "${registry}:${imageTag}"
        region     = "ap-south-2"
    }

    stages {

        stage('Clone Git Repo') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        credentialsId: '',
                        url: 'https://github.com/dubey2002/Spring-Boot-Deployment-EKS-CI-CD.git'
                    ]]
                ])
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imgName = JOB_NAME.toLowerCase()
                    sh "docker build -t ${imgName}:${imageTag} ."
                    sh "docker tag ${imgName}:${imageTag} ${fullImage}"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        mkdir -p ~/.aws
                        echo "[default]" > ~/.aws/credentials
                        echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/credentials
                        echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials
                        echo "[default]" > ~/.aws/config
                        echo "region=ap-south-2" >> ~/.aws/config

                        aws ecr get-login-password --region ap-south-2 | \
                        docker login --username AWS --password-stdin accountid.dkr.ecr.ap-south-2.amazonaws.com

                        docker push ${fullImage}
                        docker rmi ${fullImage}
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        withKubeConfig([credentialsId: 'kubeconfig', serverUrl: '']) {
                            sh '''
                                mkdir -p ~/.aws
                                echo "[default]" > ~/.aws/credentials
                                echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/credentials
                                echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials
                                echo "[default]" > ~/.aws/config
                                echo "region=ap-south-2" >> ~/.aws/config

                                curl -sLO https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl
                                chmod +x ./kubectl

                                export BUILD_ID=${BUILD_ID}
                                envsubst < eks-deploy-k8s.yaml > eks-deploy-k8s-rendered.yaml
                                ./kubectl apply -f eks-deploy-k8s-rendered.yaml
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully. App deployed to EKS!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs and slap some YAML if needed."
        }
    }
}
