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
  steps {
    script {
      // ask Jenkins for the jdk17 install path
      def jdk17Home = tool 'jdk17'
      bat """
        :: 1) force JAVA_HOME to JDK17
        set JAVA_HOME=${jdk17Home}
        set PATH=%JAVA_HOME%\\bin;%PATH%

        :: 2) download & unzip SonarScanner CLI
        curl -sSLo sonar-scanner.zip ^
          https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
        powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath scanner -Force"

        :: 3) prepend scanner/bin and run the analysis under Java 17
        set PATH=%CD%\\scanner\\sonar-scanner-4.8.0.2856-windows\\bin;%PATH%
        sonar-scanner ^
          -Dsonar.login=%SONAR_TOKEN%
      """
    }
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
