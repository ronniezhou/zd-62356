pipeline {
    agent { node { label 'labelName' } } 
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello, Maven'
                sleep 100
            }
        }
        stage('Example Test') {
            steps {
                echo 'Hello, JDK'
            }
        }
    }
}
