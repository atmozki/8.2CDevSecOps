pipeline {
  agent any

  tools {
    jdk 'jdk17'
    nodejs 'Node14'
  }

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
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
        // Ensure coverage report exists (if you added a coverage script)
        bat 'npm run coverage || exit 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // This will show known CVEs in the output
        bat 'npm audit || exit 0'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        // 1) Download & unzip the SonarScanner CLI for Windows
        bat '''
curl -sSLo sonar-scanner.zip ^
  https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath scanner"
'''

        // 2) Run the scanner, pointing at your sonar-project.properties and using the token
        bat '''
set PATH=%CD%\\scanner\\sonar-scanner-4.8.0.2856-windows\\bin;%PATH%
sonar-scanner ^
  -Dsonar.login=%SONAR_TOKEN%
'''
      }
    }
  }
}
