pipeline {
  agent any

  tools {
    // Node.js version configured in Jenkins Global Tool Configuration
    nodejs 'Node14'
  }

  stages {

    // Tool: Git
    // Purpose: Clone the project repository from GitHub
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/atmozki/8.2CDevSecOps.git'
      }
    }

    // Tool: npm
    // Purpose: Install all project dependencies
    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }

    // Tool: Mocha (test runner) via npm
    // Purpose: Run unit and functional tests; save output to log
    stage('Run Tests') {
      steps {
        // capture everything into test.log
        bat 'npm test > test.log 2>&1 || exit 0'
      }
      post {
        success {
          emailext(
            subject: "Run Tests SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to:      "dennisjojok@gmail.com",
            body:    """\
              Stage: Run Tests
              Status: SUCCESS

              See attached log for details.
            """.stripIndent(),
            mimeType: 'text/plain; charset=UTF-8',
            attachmentsPattern: 'test.log'
          )
        }
        failure {
          emailext(
            subject: "Run Tests FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to:      "dennisjojok@gmail.com",
            body:    """\
              Stage: Run Tests
              Status: FAILURE

              See attached log for details.
            """.stripIndent(),
            mimeType: 'text/plain; charset=UTF-8',
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    // Tool: nyc (Istanbul)
    // Purpose: Generate code coverage report
    stage('Generate Coverage Report') {
      steps {
        bat 'npm run coverage > coverage.log 2>&1 || exit 0'
      }
    }

    // Tool: npm audit
    // Purpose: Run security vulnerability scan on dependencies
    stage('NPM Audit (Security Scan)') {
      steps {
        bat 'npm audit > audit.log 2>&1 || exit 0'
      }
      post {
        success {
          emailext(
            subject: "Security Scan SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to:      "dennisjojok@gmail.com",
            body:    """\
              Stage: Security Scan
              Status: SUCCESS

              See attached audit.log for details.
            """.stripIndent(),
            mimeType: 'text/plain; charset=UTF-8',
            attachmentsPattern: 'audit.log'
          )
        }
        failure {
          emailext(
            subject: "Security Scan FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to:      "dennisjojok@gmail.com",
            body:    """\
              Stage: Security Scan
              Status: FAILURE

              See attached audit.log for details.
            """.stripIndent(),
            mimeType: 'text/plain; charset=UTF-8',
            attachmentsPattern: 'audit.log'
          )
        }
      }
    }
  }
}