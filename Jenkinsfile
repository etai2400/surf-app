def dockerImage

pipeline {
    agent {
        kubernetes {
            label 'promo-app'
            yamlFile 'build-pod.yaml'
            defaultContainer 'surfapp-docker-helm-build'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    environment {
        DOCKER_IMAGE = 'etai24/surf-app'
        HELM_PACKAGE = 'etai24/surf-app-chart'
        DOCKERHUB_CREDENTIALS = credentials('surf-app-id')
    }

    
    stages {
        stage('Setup') {
            steps {
                checkout scm
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.BUILD_ID = "1." + sh(returnStdout: true, script: 'echo $BUILD_NUMBER').trim()
                    } else {
                        env.BUILD_ID = "0." + sh(returnStdout: true, script: 'echo $BUILD_NUMBER').trim()
                    }
                    currentBuild.displayName += " {build-name:" + env.BUILD_ID + "}"
                }
            }
        }
        stage('Build Docker image') {
            steps {
                script {
                    dockerImage = docker.build(DOCKER_IMAGE+":"+env.BUILD_ID,"--no-cache .")
                }
            }
        }

        stage('Build Helm Chart') {
            steps {
                sh 'helm lint surf-booking-chart'
                sh 'helm package surf-booking-chart --version '+env.BUILD_ID
            }
        }
        
        stage('Push Docker image ') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'surf-app-id') {
                        dockerImage.push()
                    }
                }
            }

        }
        stage('Push HELM chart') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'surf-app-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u ${DOCKERHUB_USER} --password-stdin"
                        sh 'helm push surf-booking-chart-'+env.BUILD_ID+'.tgz oci://registry-1.docker.io/etai24'
                    }
                }
            }

        }
    }

    post{
        failure{
            script {
                echo "Job failed. Email was sent to lioratari12@gmail.com, yovelchen@gmail.com, etai2400@gmail.com"
            }
            mail to: "etai2400@gmail.com",
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
    }
}
