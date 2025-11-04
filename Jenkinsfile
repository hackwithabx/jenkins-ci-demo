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
                    dir('PROJECT_DIR') { // <-- Replace PROJECT_DIR with actual folder containing pom.xml
                        sh 'mvn clean compile'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                container(name: 'maven') {
                    dir('PROJECT_DIR') {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('StaticAnalysis') {
            steps {
                container(name: 'maven') {
                    dir('PROJECT_DIR') {
                        sh 'mvn org.owasp:dependency-check-maven:check'
                    }
                }
            }
        }

        stage('Package') {
            parallel {
                stage('CreateJarFile') {
                    steps {
                        container(name: 'maven') {
                            dir('PROJECT_DIR') {
                                sh 'mvn package -DskipTests'
                            }
                        }
                    }
                }
                stage('OCIImageBuildAndPublish') {
                    steps {
                        container(name: 'kaniko') {
                            sh '''
                            /kaniko/executor \
                            --dockerfile=`pwd`/Dockerfile \
                            --context=`pwd` \
                            --insecure \
                            --skip-tls-verify \
                            --cache=true \
                            --destination=docker.io/abxabhishekdocker/dso-demo:latest
                            '''
                        }
                    }
                }
            }
        }
    }
}
