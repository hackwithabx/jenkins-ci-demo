pipeline {
  agent {
    kubernetes {
      yamlFile 'k8s-agent-pod.yaml'
    }

  }
  stages {
    stage('Build') {
      steps {
        container(name: 'maven') {
          sh 'mvn clean compile'
        }

      }
    }

    stage('Test') {
      steps {
        container(name: 'maven') {
          sh 'mvn test'
        }

      }
    }

    stage('Package') {
      steps {
        container(name: 'maven') {
          sh 'mvn package'
        }

      }
    }

  }
}