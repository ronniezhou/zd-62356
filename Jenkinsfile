#!groovy

pipeline {

    agent any

    stages {
        stage('Collect Static Code Analysis Results') {
            when {
                anyOf { branch 'master' ; branch 'release/*' }
                not { changeRequest() }
                not { expression { return params.QA_PROMOTION } }
                not { expression { return params.EMERGENCY } }
            }
            parallel {
                stage('Collect Checkstyle Violations') {
                    steps {
                        recordIssues tool: checkStyle(pattern: '**/build/reports/checkstyle/*.xml')
                    }
                }
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
}
