pipeline {
    
    agent {
        label 'Slave1'
    }

    tools 
    {
        maven 'maven-3.8.7'
    }
    
    stages {
        stage('SCM-Checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/manju65char/star-agile-health-care.git'

            }
              post {
                failure {
                  sh "echo 'Send mail on failure'"
                  mail to:"dummyid@gmail.com", from: 'dummyid@gmail.com', subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "we failed."
                }
              }
			}
        stage('Build with Maven') {
            steps {
                sh 'mvn compile'
                sh 'mvn test'
                sh 'mvn package'
            }
        }
    }
}
