pipeline {
  agent any

  environment {
    // optional global variables (you can customize)
    APP_NAME = "myapp"
    HELM_CHART_PATH = "./myapp"
    VALUES_FILE = "./myapp/values.yaml"
  }

  stages {

    stage('Checkout Code') {
      steps {
        echo "Checking out source code..."
        checkout scm
      }
    }

    stage('Set Up Kubeconfig') {
      steps {
        echo "Setting up Kubeconfig for GKE access..."
        // Retrieve kubeconfig content from Jenkins Credentials
        withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
          sh '''
            # Write kubeconfig content to a temporary file
            echo "$KUBECONFIG_CONTENT" > kubeconfig.yaml
            export KUBECONFIG=$PWD/kubeconfig.yaml

            # Verify cluster connection
            echo "Testing cluster access..."
            kubectl cluster-info
          '''
        }
      }
    }

    stage('Helm Dependency Update') {
      steps {
        echo "Updating Helm dependencies (if any)..."
        sh 'helm dependency update ./myapp || true'
      }
    }

    stage('Deploy with Helm') {
      steps {
        echo "Deploying application using Helm..."
        withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
          sh '''
            echo "$KUBECONFIG_CONTENT" > kubeconfig.yaml
            export KUBECONFIG=$PWD/kubeconfig.yaml

            helm upgrade --install ${APP_NAME} ${HELM_CHART_PATH} \
              -f ${VALUES_FILE} \
              --wait --timeout 5m --atomic

            echo "Helm deployment completed successfully!"
          '''
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        echo "Verifying Kubernetes resources..."
        withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
          sh '''
            echo "$KUBECONFIG_CONTENT" > kubeconfig.yaml
            export KUBECONFIG=$PWD/kubeconfig.yaml

            kubectl get pods -o wide
            kubectl get svc
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment completed successfully!"
    }
    failure {
      echo "❌ Deployment failed. Check logs for details."
    }
  }
}
