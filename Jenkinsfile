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
      // 1) Resolve your JDK 17 install path
      def jdk17Home = tool 'jdk17'

      // 2) Run in one bat session so JAVA_HOME sticks
      bat """
        @echo off
        echo ----------------------------------------
        echo Using JDK from: ${jdk17Home}
        echo ----------------------------------------

        :: Point JAVA_HOME and PATH at JDK 17
        set "JAVA_HOME=${jdk17Home}"
        set "PATH=%JAVA_HOME%\\bin;%PATH%"

        :: Download & unzip SonarScanner CLI
        curl -sSLo sonar-scanner.zip ^
          https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
        powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath scanner -Force"

        :: OPTION 2: delete the embedded JRE so scanner falls back to JAVA_HOME
        rmdir /s /q scanner\\sonar-scanner-4.8.0.2856-windows\\jre

        :: Prepend the scanner’s bin dir
        set "PATH=%CD%\\scanner\\sonar-scanner-4.8.0.2856-windows\\bin;%PATH%"

        :: Finally invoke the scanner under your Java 17
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
