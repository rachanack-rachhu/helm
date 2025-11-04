pipeline {
  agent any

  environment {
    KUBECONFIG = credentials('kubeconfig-credential-id')
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Install Helm (if not present)') {
      steps {
        sh '''
          if ! command -v helm &> /dev/null; then
            echo "Installing Helm..."
            curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
            chmod 700 get_helm.sh
            ./get_helm.sh
          else
            echo "Helm is already installed."
          fi
        '''
      }
    }

    stage('Helm Dependency Update') {
      steps {
        sh 'helm dependency update ./myapp || true'
      }
    }

    stage('Deploy with Helm') {
      steps {
        sh '''
          helm upgrade --install myapp ./myapp -f ./myapp/values.yaml --wait --timeout 5m --atomic
        '''
      }
    }

    stage('Verify Deployment') {
      steps {
        sh '''
          kubectl get pods
          kubectl get svc
        '''
      }
    }
  }
}
