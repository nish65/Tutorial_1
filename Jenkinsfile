@Library('my-shared-library') _

pipeline {

    agent none

    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', defaultValue: 'javapp', description: "name of the docker build")
        string(name: 'ImageTag', defaultValue: 'v1', description: "tag of the docker build")
        string(name: 'DockerHubUser', defaultValue: 'nishwanths', description: "DockerHub username")
    }

    stages {

        stage('Git Checkout') {
            agent any
            when { expression { params.action == 'create' } }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/nish65/Tutorial_1.git"
                )
            }
        }

        stage('Maven Stages') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }
            when { expression { params.action == 'create' } }
            stages {

                stage('Unit Test') {
                    steps { script { mvnTest() } }
                }

                stage('Integration Test') {
                    steps { script { mvnIntegrationTest() } }
                }

                stage('Sonar Analysis') {
                    steps {
                        script {
                            def SonarQubecredentialsId = 'sonarqube-api'
                            statiCodeAnalysis(SonarQubecredentialsId)
                        }
                    }
                }

                stage('Quality Gate') {
                    steps {
                        script {
                            def SonarQubecredentialsId = 'sonarqube-api'
                            QualityGateStatus(SonarQubecredentialsId)
                        }
                    }
                }

                stage('Maven Build') {
                    steps { script { mvnBuild() } }
                }
            }
        }

        stage('Docker Image Build') {
            agent any
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerBuild("${params.ImageName}",
                                "${params.ImageTag}",
                                "${params.DockerHubUser}")
                }
            }
        }

        /*stage('Docker Image Scan') {
            agent any
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImageScan("${params.ImageName}",
                                    "${params.ImageTag}",
                                    "${params.DockerHubUser}")
                }
            }
        } */

        //Instead of installing trivy in the jenkins container, I been directly downloading from the docker image(very effective approach)

        /*stage('Docker Image Scan') {
    agent {
        docker {
            image 'aquasec/trivy:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    steps {
        sh "trivy image ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag}"
    }
}*/
   //Above stage was failing due to Ignore image ENTRYPOINT and Allow Jenkins to run its own shell command

        stage('Docker Image Scan') {
    agent {
        docker {
            image 'aquasec/trivy:latest'
            args '--entrypoint="" -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    steps {
        sh """
        mkdir -p .trivycache
        trivy image \
          --cache-dir .trivycache \
          ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag}
        """
    }
}

        stage('Docker Image Push') {
            agent any
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImagePush("${params.ImageName}",
                                    "${params.ImageTag}",
                                    "${params.DockerHubUser}")
                }
            }
        }

        /*stage('Docker Image Cleanup') {
            agent any
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}",
                                       "${params.ImageTag}",
                                       "${params.DockerHubUser}")
                }
            }
        }*/
    }
}
