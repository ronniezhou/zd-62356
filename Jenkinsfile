pipeline {
    agent any
    stages {
        stage('Collect Static') {
            steps {
                script{
                    jacoco inclusionPattern: '**/com/verisign/**/*'
                }
            }
        }
    }
}
