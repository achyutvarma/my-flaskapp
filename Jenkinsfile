pipeline {
  agent any

  environment {
    REMOTE = "35.89.239.129"
    REMOTE_USER = "deploy"
    DEPLOY_DIR = "/opt/apps/myapp"
    SSH_CRED = "deploy-key"   // the credential ID you added
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup & Test') {
      steps {
        sh '''
          # create venv for build/test (agent must have python3)
          python3 --version || (echo "python3 required on agent" && false)
          python3 -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
          pytest -q tests/
        '''
      }
    }

    stage('Package') {
      steps {
        sh 'zip -r myapp-${BUILD_NUMBER}.zip * -x .git* venv*'
        archiveArtifacts artifacts: "myapp-${BUILD_NUMBER}.zip", fingerprint: true
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: [env.SSH_CRED]) {
          sh """
            scp -o StrictHostKeyChecking=no myapp-${BUILD_NUMBER}.zip ${REMOTE_USER}@${REMOTE}:/tmp/
            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE} 'bash /opt/scripts/deploy_myapp.sh /tmp/myapp-${BUILD_NUMBER}.zip'
          """
        }
      }
    }
  }

  post {
    success {
      echo "Deployment finished: build ${env.BUILD_NUMBER}"
    }
    failure {
      echo "Build or deploy failed."
    }
  }
}
