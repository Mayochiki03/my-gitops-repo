pipeline {
    agent any

    environment {
        ARGOCD_SERVER = "argocd-server.argocd.svc.cluster.local"
        ARGOCD_OPTS   = "--insecure --grpc-web"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mayochiki03/my-gitops-repo.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      sonar-scanner \
                        -Dsonar.projectKey=hello-helm \
                        -Dsonar.projectName=hello-helm \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Sync ArgoCD') {
            steps {
                withCredentials([string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')]) {
                    sh '''
                      ./argocd app sync hello-helm \
                        --server $ARGOCD_SERVER \
                        --auth-token $ARGOCD_TOKEN
                    '''
                }
            }
        }
    }
}
