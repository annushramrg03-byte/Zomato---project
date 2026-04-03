pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }

    environment {
        DOCKER_IMAGE = "annushram/zomato"
        SCANNER_HOME = tool 'Sonarqube-server'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/annushramrg03-byte/Zomato---project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t zomato .'
                }
            }
        }

        stage('Tag & Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
                        sh '''
                            docker tag zomato ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv([
                    'AWS_DEFAULT_REGION=ap-south-1',
                    'KUBECONFIG=/var/lib/jenkins/.kube/config'
                ]) {
                    sh '''
                        kubectl get nodes
                        kubectl apply -f Kubernetes/deployment.yaml
                        kubectl apply -f Kubernetes/service.yaml
                        kubectl apply -f Kubernetes/hpa.yml
                        kubectl rollout status deployment/zomato
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully. App deployed to Kubernetes.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
