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
