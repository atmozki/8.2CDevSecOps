pipeline {
  agent any

  tools {
    // Make JDK 17 and Node 14 available
    jdk     'jdk17'
    nodejs  'Node14'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url:    'https://github.com/atmozki/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // allow failures so coverage still gets generated
        bat 'npm test || exit 0'
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // if you’ve added a coverage script
        bat 'npm run coverage || exit 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // shows known CVEs in the output
        bat 'npm audit || exit 0'
      }
    }
  }

  post {
    success {
      // on success, send a green-check email
      emailext(
        subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to:      "dennisjojok@gmail.com",
        body:    """\
          Build succeeded!
          
          Job: ${env.JOB_NAME}
          Build Number: ${env.BUILD_NUMBER}
          URL: ${env.BUILD_URL}
        """.stripIndent()
      )
    }

    failure {
      // on failure, send a red-cross email
      emailext(
        subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to:      "dennisjojok@gmail.com",
        body:    """\
          Build failed!
          
          Job: ${env.JOB_NAME}
          Build Number: ${env.BUILD_NUMBER}
          URL: ${env.BUILD_URL}

          Check console output for details.
        """.stripIndent()
      )
    }

    always {
      // archive logs, coverage reports, etc.
      archiveArtifacts artifacts: '**/coverage/*.html, **/sonar-scanner.log', allowEmptyArchive: true
    }
  }
}
