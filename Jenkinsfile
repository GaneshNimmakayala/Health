pipeline {
    agent { label 'Health_slave' }

    environment {    
        DOCKERHUB_CREDENTIALS = credentials('Health_care')
    }

    stages {
        stage('SCM_Checkout') {
            steps {
                echo "Perform SCM Checkout"
                git 'https://github.com/GaneshNimmakayala/Health.git'               
            }
        }
        stage('Application Build') {
            steps {
                echo "Perform Application Build"
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker version'
                sh "docker build -t ganeshnimmakayala/health-application:${BUILD_NUMBER} ."
                sh 'docker image list'
                sh "docker tag ganeshnimmakayala/health-application:${BUILD_NUMBER} ganeshnimmakayala/health-application:latest"
            }
        }
        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Publish to Docker Registry') {
            steps {
                sh "docker push ganeshnimmakayala/health-application:latest"
            }
        }
        stage('Update deployment.yaml') {
            steps {
                script {
                    sh '''
                    sed -i 's|image: ganeshnimmakayala/health-application:[^ ]*|image: ganeshnimmakayala/health-application:latest|' kubernetesdeploy.yaml
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'Kubernetes', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                }
            }
        }
    }
}
