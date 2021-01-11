pipeline {
    agent any
    stages {
        stage('Collect Static') {
            steps {
                jacoco inclusionPattern: '**/com/verisign/**/*'
            }
        }
    }
}
