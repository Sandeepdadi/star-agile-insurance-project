pipeline {
    agent any

    tools {
        maven 'Maven-3'   // must match Jenkins Global Tool name
        jdk 'jdk17'       // must match Jenkins Global Tool name
    }

    environment {
        DOCKER_IMAGE = "sandeepdadi/insure-me:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Sandeepdadi/star-agile-insurance-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }


stage('Publish Test Reports') {
    steps {
        publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'Test Report'
        ])
    }
}


        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }



stage('Deploy to Test Server') {
    steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'test-server-ssh', keyFileVariable: 'SSH_KEY')]) {
            sh """
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@13.60.219.69 \
                'docker stop insureme || true && docker rm insureme || true && docker pull sandeepdadi/insure-me:10 && docker run -d -p 8080:8081 --name insureme sandeepdadi/insure-me:10'
            """
        }
    }
}


    }
}

