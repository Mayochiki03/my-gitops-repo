pipeline {
    agent any
    environment {
        ARGOCD_SERVER = "argocd.your-domain.com"  // เปลี่ยนเป็น URL ArgoCD จริง
    }
    stages {
        stage('Checkout Git') {
            steps {
                git(
                    url: 'https://github.com/Mayochiki03/my-gitops-repo.git',
                    credentialsId: 'github-mayochiki-token',
                    branch: 'main'
                )
            }
        }
        stage('Sync ArgoCD') {
            steps {
                withCredentials([string(credentialsId: 'argo_new', variable: 'ARGOCD_TOKEN')]) {
                    sh '''
                        argocd app sync my-app \
                            --server $ARGOCD_SERVER \
                            --auth-token $ARGOCD_TOKEN
                    '''
                }
            }
        }
    }
}