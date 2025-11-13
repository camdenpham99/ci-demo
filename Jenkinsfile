pipeline {
  agent any
  environment {
    WEBEX_TOKEN = credentials('WEBEX_BOT_TOKEN')
    WEBEX_ROOM  = credentials('WEBEX_ROOM_ID')
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Install') {
      steps {
        sh '''
          python3 -m venv .venv
          . .venv/bin/activate
          pip install -r requirements.txt
        '''
      }
    }
    stage('Test') {
      steps {
        sh '''
          . .venv/bin/activate
          pytest -q --junitxml=test-results.xml
        '''
        junit allowEmptyResults: false, testResults: 'test-results.xml'
      }
    }
  }
  post {
    success {
      sh """
        curl -s -X POST https://webexapis.com/v1/messages \
          -H 'Authorization: Bearer ${WEBEX_TOKEN}' \
          -H 'Content-Type: application/json' \
          -d '{ "roomId": "${WEBEX_ROOM}", "markdown": "✅ Build SUCCESS: ${JOB_NAME} #${BUILD_NUMBER} (${BUILD_URL})" }'
      """
    }
    failure {
      sh """
        curl -s -X POST https://webexapis.com/v1/messages \
          -H 'Authorization: Bearer ${WEBEX_TOKEN}' \
          -H 'Content-Type: application/json' \
          -d '{ "roomId": "${WEBEX_ROOM}", "markdown": "❌ Build FAILED: ${JOB_NAME} #${BUILD_NUMBER} (${BUILD_URL})" }'
      """
    }
  }
}
