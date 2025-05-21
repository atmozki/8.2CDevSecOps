pipeline {
  agent any

  tools {
    nodejs 'Node14'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/atmozki/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // allow failures so we still generate coverage
        bat 'npm test || exit 0'
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure coverage report exists
        bat 'npm run coverage || exit 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // This will show known CVEs in the output
        bat 'npm audit || exit 0'
      }
    }
  }

  post {
    success {
      emailext(
        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to:      "dennisjojok@gmail.com",
        body:    """\
          Hi team,

          The build *${env.JOB_NAME}* #${env.BUILD_NUMBER} succeeded!

          ▶️ ${env.BUILD_URL}

          Regards,
          Jenkins
        """.stripIndent()
      )
    }

    failure {
      emailext(
        subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to:      "dennisjojok@gmail.com",
        body:    """\
          Hi team,

          The build *${env.JOB_NAME}* #${env.BUILD_NUMBER} failed.

          Please check the console output for details:
          ▶️ ${env.BUILD_URL}

          Regards,
          Jenkins
        """.stripIndent()
      )
    }

    always {
      // archive coverage reports if any
      archiveArtifacts artifacts: '**/coverage/*.html', allowEmptyArchive: true
    }
  }
}
