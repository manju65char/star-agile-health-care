pipeline {
    agent { label 'Slave1' }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven-3.8.7"
    }

    environment {	
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    } 
    
    stages {
        stage('SCM Checkout') {
            steps {
                // Get some code from a GitHub repository
                git url: 'https://github.com/manju65char/star-agile-health-care.git'
            }
        }
        stage('Maven Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage("Docker build") {
            steps {
                sh 'docker version'
                sh "docker build -t manjunathachar/healthcare_app:${BUILD_NUMBER} ."
                sh 'docker image list'
                sh "docker tag manjunathachar/healthcare_app:${BUILD_NUMBER} manjunathachar/healthcare_app:latest"
            }
        }
        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Approve - push Image to Docker Hub') {
            steps {
                //----------------send an approval prompt-------------
                script {
                   env.APPROVED_DEPLOY_DOCKERHUB = input message: 'User input required. Choose "Yes" to approve or "Abort" to reject the Docker Hub deployment', ok: 'Yes', submitterParameter: 'APPROVER'
                }
                //-----------------end approval prompt------------
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    if(env.APPROVED_DEPLOY_DOCKERHUB == 'Yes') {
                        sh "docker push manjunathachar/healthcare_app:latest"
                    } else {
                        error('Docker Hub deployment not approved.')
                    }
                }
            }
        }
        stage('Approve - Deployment to Kubernetes Cluster') {
            steps {
                //----------------send an approval prompt-----------
                script {
                   env.APPROVED_DEPLOY_KUBERNETES = input message: 'User input required. Choose "Yes" to approve or "Abort" to reject the Kubernetes deployment', ok: 'Yes', submitterParameter: 'APPROVER'
                }
                //-----------------end approval prompt------------
            }
        }
        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    if(env.APPROVED_DEPLOY_KUBERNETES == 'Yes') {
                        sshPublisher(publishers: [sshPublisherDesc(configName: 'kube_masternode', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f k8sdeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                    } else {
                        error('Kubernetes deployment not approved.')
                    }
                }
            }
        }
    }
}
