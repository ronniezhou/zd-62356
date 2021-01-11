#!groovy

pipeline {

    agent any

    stages {
        stage('Collect Static Code Analysis Results') {
                stage('Collect All Other SCA Metrics') {
                    steps {
                        junit '**/build/test-results/**/*.xml'
                        jacoco(inclusionPattern: '**/com/verisign/**/*')
                        recordIssues tool: spotBugs(pattern: '**/build/reports/spotbugs/*.xml')
                        recordIssues tool: pmdParser(pattern: '**/build/reports/pmd/*.xml')
                    }
                }
        }
    }
}
