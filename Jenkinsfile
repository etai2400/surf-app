pipeline {
    agent {
        kubernetes {
            label 'promo-app'
            yamlFile 'build-pod.yaml'
            defaultContainer 'surfapp-docker-helm-build'
        }
    }
  
    stages {
        stage( 'Build image' ) {
            steps{
                echo 'starting to build docker image'
                sh "docker build -t surfapp-docker . "

            }
        }    
        stage ('Test'){
            steps {
                echo 'running a python unit test'
                sh 'python unit-test.py'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }    
    post{
        failure{
            script {
                echo "Job failed. Email was sent to lioratari12@gmail.com, yovelchen@gmail.com, etai2400@gmail.com"
            }
            mail to: "lioratari12@gmail.com, yovelchen@gmail.com, etai2400@gmail.com",
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
    }
}
