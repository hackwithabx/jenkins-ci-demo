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
            parallel {
                stage('CreateJarfile') {
                    steps {
                        container(name: 'maven') {
                            sh 'mvn package -DskipTests'
                        }
                    }
                }
                stage('OCIImageBnP') {
                    steps {
                        container(name: 'kaniko') {
                            sh '''
                            /kaniko/executor \
                            --dockerfile=`pwd`/Dockerfile \
                            --context=`pwd` \
                            --insecure \
                            --skip-tls-verify \
                            --cache=true \
                            --destination=docker.io/abxabhishekdocker/dso-demo
                            '''
                        }
                    }
                }
            }
        }
    }
}
