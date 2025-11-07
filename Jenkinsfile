pipeline {
    agent {
        kubernetes {
            yamlFile 'k8s-agent-pod.yaml'
        }
    }

    environment {
        REGISTRY = "gcr.io"
        IMAGE_NAME = "jenkins-ci-demo"
        TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    dir('app') {
                        sh 'mvn clean compile'
                    }
                }
            }
        }

        stage('StaticAnalysis') {
            stages {

                stage('SCA') {
                    steps {
                        container('maven') {
                            dir('app') {
                                sh 'mvn org.owasp:dependency-check-maven:check'
                            }
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'app/target/dependency-check-report.html',
                                              allowEmptyArchive: true
                        }
                    }
                }

                stage('GenerateSBOM') {
                    steps {
                        container('maven') {
                            dir('app') {
                                sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
                            }
                        }
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'app/target/bom.xml',
                                              allowEmptyArchive: true
                        }
                    }
                }

                stage('OSS License Checker') {
                    steps {
                        dir('doc') {
                            sh 'license_finder'
                        }
                    }
                }

            }
        }

        /* ✅ ✅ NEW SAST STAGE — REQUIRED BY LAB MANUAL */
        stage('SAST') {
            steps {
                container('slscan') {

                    dir('app') {
                        sh '''
                            echo "Running SAST + Dependency Scan using ShiftLeft SCAN..."
                            scan --type java,depscan --build
                        '''
                    }

                }
            }
            post {
                success {
                    archiveArtifacts allowEmptyArchive: true,
                                     artifacts: 'app/reports/*',
                                     fingerprint: true,
                                     onlyIfSuccessful: true
                }
            }
        }

        stage('Package') {
            parallel {

                stage('CreateJarFile') {
                    steps {
                        container('maven') {
                            dir('app') {
                                sh 'mvn -DskipTests package'
                            }
                        }
                    }
                }

                stage('OCIImageBuildAndPublish') {
                    steps {
                        container('kaniko') {
                            sh '''
                                /kaniko/executor \
                                  --context ${WORKSPACE}/app \
                                  --dockerfile ${WORKSPACE}/app/Dockerfile \
                                  --destination ${REGISTRY}/${IMAGE_NAME}:${TAG}
                            '''
                        }
                    }
                }

            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
    }
}
