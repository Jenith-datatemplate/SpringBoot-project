pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jenithdt/spring_project:${BUILD_NUMBER}"
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['build', 'deploy', 'remove'],
            description: 'Select what you want to do'
        )
    }

    stages {

        stage('Checkout Code') {
            when { expression { params.ACTION == 'build' } }
            steps {
                git url: 'https://github.com/Jenith-datatemplate/SpringBoot-project.git',
                    branch: 'master'
            }
        }

        stage('Build Docker Image') {
            when { expression { params.ACTION == 'build' } }
            steps {
                 sh '''
                docker build -t $DOCKER_IMAGE .
                docker tag $DOCKER_IMAGE jenithdt/static-webpage:latest
                '''
            }
        }

        stage('Docker Login') {
            when { expression { params.ACTION == 'build' } }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-2008',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            when { expression { params.ACTION == 'build' } }
            steps {
                sh '''
                docker push $DOCKER_IMAGE
                docker push jenithdt/static-webpage:latest
                '''
            }
        }

        stage('Deploy Application') {
            when { expression { params.ACTION == 'build' } }
            steps {
                sh '''
                docker-compose down || true
                docker-compose pull
                docker-compose up -d
                '''
            }
        }

        stage('Remove Application') {
            when { expression { params.ACTION == 'remove' } }
            steps {
                sh 'docker-compose down || true'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            echo 'Pipeline completed'
        }
    }
}
