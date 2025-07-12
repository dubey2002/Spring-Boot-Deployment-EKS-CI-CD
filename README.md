🧠 Project Overview
      This project demonstrates a complete CI/CD pipeline for a Spring Boot application:
      
      Built using Maven
      
      Containerized with Docker
      
      Pushed to AWS ECR
      
      Deployed to AWS EKS
      
      CI/CD pipeline managed using Jenkins

🔧 Prerequisites
        Ensure the following components are ready before running the pipeline:
        
        🧱 Jenkins Setup
        Jenkins installed and running
        
        A Jenkins agent (slave) with:
        
        ✅ Docker
        
        ✅ Maven
        
        ✅ AWS CLI
        
        ✅ kubectl (or downloaded in pipeline)

Required Jenkins Plugins:

            Pipeline
            
            Git
            
            Credentials Binding
            
            Kubernetes CLI Plugin (for withKubeConfig)
            
            (Optional) AWS Credentials Plugin for simplified AWS access handling

🔐 Jenkins Credentials Required
            Go to: Manage Jenkins → Credentials → Global
            Create the following credentials:

ID	Type	Description
            aws-access-key-id	Secret Text	Your AWS access key ID
            aws-secret-access-key	Secret Text	Your AWS secret access key
            kubeconfig	File	Your EKS kubeconfig file

☁️ AWS & ECR Setup
            Create an ECR Repository
            aws ecr create-repository --repository-name three-tier --region ap-south-2
            Ensure your IAM user/role has permissions for:
            
            ECR: ecr:BatchCheckLayerAvailability, ecr:GetDownloadUrlForLayer, ecr:PutImage, etc.
            
            EKS: eks:DescribeCluster, eks:ListClusters, etc.
            
            Make sure the EKS cluster is created and reachable from Jenkins.

📁 Project Structure

      spring-boot-app/
      ├── Dockerfile                 # Container definition
      ├── eks-deploy-k8s.yaml        # Kubernetes Deployment & Service
      ├── Jenkinsfile                # CI/CD pipeline config
      ├── pom.xml                    # Maven config
      └── src/                       # Spring Boot source code
