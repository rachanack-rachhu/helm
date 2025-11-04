pipeline {
  agent any

  environment {
    // optional: define any environment variables
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
        // 🟢 This part is the fix: decode kubeconfig from Jenkins credentials
        withCredentials([string(credentialsId: 'gke-kubeconfig-b64', variable: 'KUBECONFIG_B64')]) {
          sh '''
            echo "$KUBECONFIG_B64" | base64 -d > kubeconfig
            export KUBECONFIG=$(pwd)/kubeconfig

            echo "🔹 Verifying Kubernetes connection..."
            kubectl config current-context
            kubectl get nodes

            echo "🔹 Deploying Helm chart..."
            helm upgrade --install myapp ./myapp -f ./myapp/values.yaml --wait --timeout 5m --atomic
          '''
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        sh '''
          export KUBECONFIG=$(pwd)/kubeconfig
          echo "🔹 Checking deployed resources..."
          kubectl get pods
          kubectl get svc
        '''
      }
    }
  }
}
