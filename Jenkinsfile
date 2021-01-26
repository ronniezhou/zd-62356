pipeline {
    agent any 
    stages {
        stage('Example Build') {
            steps {
                println currentBuild.getBuildCauses()
                echo 'Hello, Maven'
            }
        }
        stage('Example Test') {
            steps {
                echo 'Hello, JDK'
                sh "touch readme.txt"
            }
        }
    }
}
