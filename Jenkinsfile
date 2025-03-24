pipeline {
    agent any

    tools {
        jdk 'jdk'       
        nodejs 'node'  
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        EKS_CLUSTER_NAME = "hotstar-cluster"
        AWS_REGION = "ap-south-1"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mohitbisht1997/HotStar-clone'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  
                    sh '''
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=Hotstar \
                    -Dsonar.projectKey=Hotstar \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Node.js Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy FS scan') {
            steps {
                script {
                    sh 'trivy fs --severity HIGH,CRITICAL ./ --format table --output trivy-fs-report.txt || true'
                }
            }
        }

        stage('Docker build and push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '351be11d-c448-4198-8338-6e887678688c') {
                        sh 'docker build -t mohit993/hotstar:latest .'
                        sh 'docker push mohit993/hotstar:latest'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh 'trivy image --severity HIGH,CRITICAL mohit993/hotstar:latest --format table --output trivy-image-report.txt || true'
                }
            }
        }

        stage('Configure AWS and EKS') {
            steps {
                sh "aws sts get-caller-identity"
                sh "aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION"
                
            }
        }    
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'hotstar-cluster', contextName: '', credentialsId: 'k8s-credentials', namespace: 'hotstar', restrictKubeConfigAccess: false, serverUrl: 'https://C292684A50F08EE5C93A743900734CE1.sk1.ap-south-1.eks.amazonaws.com') {
                    sh '''
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }

        stage('Verify Deployment and Service') {
            steps {
                sh "kubectl get pods -n hotstar"
                sh "kubectl get svc -n hotstar"
            }
        }
    }
}
