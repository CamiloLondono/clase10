pipeline {
  agent any

  environment {
    SONARQUBE_SERVER = 'sonarqube'
    SONAR_SCANNER_HOME = '/var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner'
    VENV = "${WORKSPACE}\\venv"
  }


  stages {
   
   
   stage('Check Python Version') {
      steps {
      bat '"C:\\Python313\\python.exe"--version'
    }
  }



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
  }
}
