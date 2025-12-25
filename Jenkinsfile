pipeline {
    agent any

    environment {
        ARGOCD_SERVER = "argocd-server.argocd.svc.cluster.local"
        ARGOCD_OPTS   = "--insecure --grpc-web"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mayochiki03/my-gitops-repo.git',
                    branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      ${scannerHome}/bin/sonar-scanner \
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

        stage('Install ArgoCD CLI') {
            steps {
                sh '''
                  if [ ! -f ./argocd ]; then
                    curl -fsSL -o argocd \
                      https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
                    chmod +x argocd
                  fi
                '''
            }
        }

        stage('Sync ArgoCD') {
            steps {
                withCredentials([
                    string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')
                ]) {
                    sh '''
                      ./argocd app sync hello-helm \
                        --server ${ARGOCD_SERVER} \
                        --auth-token ${ARGOCD_TOKEN} \
                        ${ARGOCD_OPTS}
                    '''
                }
            }
        }
    }
}
