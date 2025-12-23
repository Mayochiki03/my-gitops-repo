pipeline {
    agent any

    environment {
        ARGOCD_SERVER = "argocd-server.argocd.svc.cluster.local"
    }

    stages {

        stage('Checkout Git') {
            steps {
                git(
                    url: 'https://github.com/Mayochiki03/my-gitops-repo.git',
                    branch: 'main'
                )
            }
        }

        stage('Install ArgoCD CLI') {
            steps {
                sh '''
                  if [ ! -f ./argocd ]; then
                    echo "Installing argocd CLI locally..."
                    curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
                    chmod +x argocd
                  fi

                  ./argocd version
                '''
            }
        }

        stage('Sync ArgoCD') {
            steps {
                withCredentials([string(credentialsId: 'argo_new', variable: 'ARGOCD_TOKEN')]) {
                    sh '''
                      ./argocd app sync hello-helm \
                        --server $ARGOCD_SERVER \
                        --auth-token $ARGOCD_TOKEN \
                        --insecure
                    '''
                }
            }
        }
    }
}
