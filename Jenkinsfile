pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub')
  }
  stages {
    stage('Maven Compile') {
      steps {
        sh 'mvn compile'
      }
    }
    stage('Test JUnit') {
      steps {
        sh 'mvn test -Dtest=PisteServicesImplTest'
      }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install -DskipTests'
      }
    }
    stage('SonarQube') {
      steps {
        sh 'mvn sonar:sonar -Dsonar.login=admin -Dsonar.password=sonar'
      }
    }
    stage('Nexus Deployment') {
      steps {
        sh 'mvn deploy -DskipTests'
      }
    }
    stage('Building Image') {
      steps {
        sh 'docker build -t jihedmrouki/ski .'
      }
    }
    stage('Pushing Image to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/ski:latest'
      }
    }
    stage('Docker Compose') {
            steps {
                sh 'docker compose up -d'
            }
        }
    stage('Cleanup Dockerhub logout') {
      steps {
        sh 'docker rmi -f $DOCKERHUB_CREDENTIALS_USR/ski:latest'
        sh 'docker logout'
      }
    }
  }
  post {
        always {
            emailext (
                to: 'mrouki.jihed@esprit.tn',
                subject: "Pipeline ${currentBuild.fullDisplayName} completed",
                body: "The pipeline ended with result: ${currentBuild.result}",
                mimeType: 'text/plain'
            )
        }
    }
}