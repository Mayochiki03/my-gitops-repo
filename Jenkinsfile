pipeline {
    agent any

    environment {
        // ===== Argo CD =====
        ARGOCD_SERVER = "argocd-server.argocd.svc.cluster.local"
        ARGOCD_OPTS   = "--insecure --grpc-web"

        // ===== Sonar =====
        SONAR_PROJECT_KEY  = "hello-helm"
        SONAR_PROJECT_NAME = "hello-helm"
    }

    stages {

        /* =========================
           1. Checkout
           ========================= */
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mayochiki03/my-gitops-repo.git',
                    branch: 'main'
            }
        }

        /* =========================
           2. Install SonarScanner
           ========================= */
        stage('Install SonarScanner') {
            steps {
                sh '''
                  set -e

                  if [ ! -d sonar-scanner ]; then
                    echo "Installing SonarScanner..."
                    curl -fsSL \
                      https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.tar.gz \
                      | tar -xz
                    mv sonar-scanner-* sonar-scanner
                  fi

                  export PATH=$PWD/sonar-scanner/bin:$PATH
                  sonar-scanner --version
                '''
            }
        }

        /* =========================
           3. SonarQube Analysis
           ========================= */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      set -e
                      export PATH=$PWD/sonar-scanner/bin:$PATH

                      sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        /* =========================
           4. Quality Gate
           ========================= */
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* =========================
           5. Install ArgoCD CLI
           ========================= */
        stage('Install ArgoCD CLI') {
            steps {
                sh '''
                  set -e

                  if [ ! -f ./argocd ]; then
                    echo "Installing ArgoCD CLI..."
                    curl -fsSL -o argocd \
                      https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
                    chmod +x argocd
                  fi

                  ./argocd version --client
                '''
            }
        }

        /* =========================
           6. Sync ArgoCD
           ========================= */
        stage('Sync ArgoCD') {
            steps {
                withCredentials([
                    string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')
                ]) {
                    sh '''
                      set -e

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
