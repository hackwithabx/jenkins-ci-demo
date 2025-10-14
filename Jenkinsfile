pipeline {
    agent {
        kubernetes {
            yamlFile 'k8s-agent-pod.yaml'
        }
    }

    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    sh 'mvn test'
                }
            }
        }

        stage('Package') {
            steps {
                container('maven') {
                    sh 'mvn package'
                }
            }
        }
    }
}
