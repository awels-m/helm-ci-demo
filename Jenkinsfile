pipeline {
  agent any

  environment {
    // 1) Full paths to the binaries (Apple Silicon Homebrew):
    HELM_BIN    = '/opt/homebrew/bin/helm'     // run `which helm` to confirm
    KUBECTL_BIN = '/opt/homebrew/bin/kubectl'  // run `which kubectl` to confirm

    // 2) Tell Jenkins exactly which kubeconfig to use (YOUR user’s file)
    KUBECONFIG  = "${HOME}/.kube/config"

    // 3) (Optional but helpful) Ensure /opt/homebrew/bin is on PATH inside Jenkins
    PATH = "/opt/homebrew/bin:${PATH}"

    // 4) Chart/deploy settings
    KUBE_NAMESPACE = 'default'
    RELEASE_NAME   = 'my-webapp'
    CHART_DIR      = './webapp'
  }

  options {
    // safe to leave only timestamps (ansiColor needs a plugin)
    timestamps()
  }

  triggers {
    // polling for commits; replace with webhook later
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
        sh 'echo "Helm at:    $HELM_BIN"'
        sh 'echo "Kubectl at: $KUBECTL_BIN"'
        sh '$HELM_BIN version'
        sh '$KUBECTL_BIN version --client'
        sh 'echo "KUBECONFIG=$KUBECONFIG"'
        sh 'ls -l "$KUBECONFIG"'
        sh '$KUBECTL_BIN config current-context || true'
        // If you know you’re using kind named jx, you can force it:
        // sh '$KUBECTL_BIN config use-context kind-jx'
        sh '$KUBECTL_BIN get ns'
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
          $HELM_BIN upgrade --install "$RELEASE_NAME" "$CHART_DIR" \
            --namespace "$KUBE_NAMESPACE" \
            --wait --timeout 5m
        '''
      }
    }

    stage('Post-deploy checks') {
      steps {
        sh '$KUBECTL_BIN -n $KUBE_NAMESPACE get deploy,po,svc'
        sh '$KUBECTL_BIN -n $KUBE_NAMESPACE rollout status deploy/$RELEASE_NAME'
      }
    }
  }

  post {
    success { echo '✅ Deployment succeeded.' }
    failure { echo '❌ Deployment failed. Check the logs above.' }
  }
}

