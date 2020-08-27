pipeline {
    agent none 
    stages {
        stage('Example Build') {
            steps {
                println currentBuild.getBuildCauses()
                echo 'Hello, Maven1'
            }
        }
        stage('Example Test') {
            steps {
                echo 'Hello, JDK'
            }
        }
    }
}
