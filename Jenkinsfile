pipeline {
  agent any

  environment {
    // Absolute path to Helm binary on Apple Silicon (Homebrew):
    HELM_BIN = '/opt/homebrew/bin/helm'   // <-- run `which helm` to confirm
    KUBE_NAMESPACE = 'default'
    RELEASE_NAME   = 'my-webapp'
    CHART_DIR      = './webapp'
  }

  options {
    ansiColor('xterm')
    timestamps()
  }

  triggers {
    // Poll for new commits every ~2 minutes. Replace with webhook later.
    pollSCM('H/2 * * * *')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Tooling sanity') {
      steps {
        sh 'echo "Helm at: $HELM_BIN"'
        sh '$HELM_BIN version'
        sh 'kubectl version --client'
        sh 'kubectl config current-context'
        sh 'kubectl get ns'
      }
    }

    stage('Helm Lint') {
      steps {
        sh '$HELM_BIN lint $CHART_DIR'
      }
    }

    stage('Deploy with Helm') {
      steps {
        sh '''
          set -euxo pipefail
          # upgrade --install = create if not exists, otherwise update
          # --wait makes Jenkins fail the build if rollout cannot become Ready
          $HELM_BIN upgrade --install "$RELEASE_NAME" "$CHART_DIR" \
            --namespace "$KUBE_NAMESPACE" \
            --wait --timeout 5m
        '''
      }
    }

    stage('Post-deploy checks') {
      steps {
        sh 'kubectl -n $KUBE_NAMESPACE get deploy,po,svc'
        sh 'kubectl -n $KUBE_NAMESPACE rollout status deploy/$RELEASE_NAME'
      }
    }
  }

  post {
    success { echo '✅ Deployment succeeded.' }
    failure { echo '❌ Deployment failed. Check the logs above.' }
  }
}

