pipeline {
  agent any

  tools {
    // Asegúrate de tener configurado el tool 'Python3' en Jenkins
    python 'Python3'
  }

  environment {
    SONARQUBE_SERVER = 'sonarqube'
    SONAR_SCANNER_HOME = '/var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner'
    VENV = "${WORKSPACE}\\venv"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/CamiloLondono/clase10'
      }
    }

    stage('Instalar dependencias') {
      steps {
        bat """
          python -m venv %VENV%
          %VENV%\\Scripts\\pip.exe install --upgrade pip
          %VENV%\\Scripts\\pip.exe install -r requirements.txt
        """
      }
    }

    stage('Pruebas unitarias') {
      steps {
        bat """
          %VENV%\\Scripts\\pytest.exe --maxfail=1 --disable-warnings --quiet
        """
      }
    }

    stage('Construir imagen Docker') {
      steps {
        script {
          dockerImage = docker.build("vladcamilo/clase10-taller")
        }
      }
    }

    stage('Login Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          bat """
            echo %DOCKER_PASS% | docker login -u %DOCKER_USER% -p %DOCKER_PASS%
          """
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.image('vladcamilo/calse10-taller').push()
        }
      }
    }

    stage('Lint') {
      steps {
        bat """
          %VENV%\\Scripts\\pip.exe install pylint
          %VENV%\\Scripts\\pylint.exe src --output-format=json > pylint-report.json || exit 0
        """
      }
    }

    stage('Análisis SonarQube') {
      steps {
        // withSonarQubeEnv("${SONARQUBE_SERVER}") {
        //   withCredentials([string(credentialsId: 'sonarqube_auth_token', variable: 'SONAR_TOKEN')]) {
        //     bat """
        //       ${SONAR_SCANNER_HOME}\\bin\\sonar-scanner.bat ^
        //         -Dsonar.projectKey=my-python-app ^
        //         -Dsonar.sources=src ^
        //         -Dsonar.host.url=http://sonarqube:9000 ^
        //         -Dsonar.token=%SONAR_TOKEN% ^
        //         -Dsonar.python.coverage.reportPaths=coverage.xml ^
        //         -Dsonar.python.pylint.reportPaths=pylint-report.json
        //     """
        //   }
        // }
      }
    }
  }
}
