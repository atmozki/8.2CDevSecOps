pipeline {
  agent any

  tools {
    // Make jdk17 and Node14 available
    jdk  'jdk17'
    nodejs 'Node14'
  }

  environment {
    // Inject your SonarCloud token
    SONAR_TOKEN = credentials('SONAR_TOKEN')
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

    stage('SonarCloud Analysis') {
      environment {
        // Resolve the jdk17 tool installation path
        JAVA_HOME = tool name: 'jdk17', type: 'jdk'
        // Ensure Java 17’s bin directory is first on PATH
        PATH      = "${JAVA_HOME}\\bin;${env.PATH}"
      }
      steps {
        // 1) Download & unzip the SonarScanner CLI
        bat '''
curl -sSLo sonar-scanner.zip ^
  https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath scanner -Force"
'''

        // 2) Run the scanner under Java 17
        bat '''
set PATH=%CD%\\scanner\\sonar-scanner-4.8.0.2856-windows\\bin;%PATH%
sonar-scanner ^
  -Dsonar.login=%SONAR_TOKEN%
'''
      }
    }
  }

  post {
    always {
      // archive logs, coverage reports, etc.
      archiveArtifacts artifacts: '**/coverage/*.html, **/sonar-scanner.log', allowEmptyArchive: true
    }
  }
}
