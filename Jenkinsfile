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
      echo "Resolved JDK17 home to: ${jdk17Home}"

      // 2) Execute everything in one bat session
      bat """
        @echo off
        echo ----------------------------------------
        echo Setting JAVA_HOME to %JAVA_HOME%
        echo ----------------------------------------

        :: Use the tool path
        set "JAVA_HOME=${jdk17Home}"
        set "PATH=%JAVA_HOME%\\bin;%PATH%"

        :: 3) Verify that java.exe is actually callable
        echo Verifying Java version…
        "%JAVA_HOME%\\bin\\java.exe" -version || (
          echo.
          echo ************************************************************
          echo ERROR: Failed to run "%JAVA_HOME%\\bin\\java.exe"
          echo ************************************************************
          exit /b 1
        )
        echo.

        :: 4) Download & unzip SonarScanner CLI
        curl -sSLo sonar-scanner.zip ^
          https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
        powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath scanner -Force"

        :: 5) OPTION 2: remove the embedded JRE so it falls back to our JAVA_HOME
        rmdir /s /q "scanner\\sonar-scanner-4.8.0.2856-windows\\jre"

        :: 6) Prepend the scanner’s bin dir
        set "PATH=%CD%\\scanner\\sonar-scanner-4.8.0.2856-windows\\bin;%PATH%"

        :: 7) Finally invoke the scanner under Java 17
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
