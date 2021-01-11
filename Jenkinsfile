#!groovy

pipeline {

    agent any

    parameters {
        booleanParam(name: 'QA_PROMOTION', defaultValue: false, description: 'Set this flag for ALL QA-Promotion jobs')
        string(name: 'RELEASE', defaultValue: '', description: 'Specify the Jenkins build id of the artifacts to be published')
        booleanParam(name: 'EMERGENCY', defaultValue: true, description: 'Set this flag to skip SCA for a release build')
    }

    environment {
        GRADLE_USER_HOME="${WORKSPACE}/.ctld_gradle_home"
        GRADLE_OPTS="-Dorg.gradle.daemon=false -Dorg.gradle.internal.http.socketTimeout=60000 -Dorg.gradle.internal.http.connectionTimeout=60000"
        ARTIFACTORY_CREDS=credentials('docker-deploy')

        VERSION = "8.4.0"
        THE_RELEASE = "${params.RELEASE ?: BUILD_NUMBER}"
        WEBEX_ALERTS_URL="https://botworkflows.webex.com/embed/run/99602d23ced172ed46fc77"
        // git rev-parse <branch-name>
        PREVIOUS_RELEASE = "97de41f53bfc2d4822760a059f96efa68909f9d3"

        // During development, use this to determine which artifacts to publish
        INITIAL_COMMIT = "${PREVIOUS_RELEASE}"
        // When regression testing commences, use this to determine which artifacts to publish
        //INITIAL_COMMIT = "${env.GIT_PREVIOUS_SUCCESSFUL_COMMIT ?: PREVIOUS_RELEASE}"

        // this is meant to get applied to feature branches and PRs
        //DASH_X_ARGS = '-x checkstyleMain -x checkstyleTest -x checkstyleIntegrationTest -x pmdMain -x pmdTest -x pmdIntegrationTest'
        DASH_X_ARGS = '-PskipSCA'

        ADDITIONAL_ARGS = ' '
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
        disableConcurrentBuilds()
        timestamps()
    }

    stages {
        stage('Release Configuration') {
            steps {
                script {
                    // for master and release branches, enable SCA by dropping the -x args
                    DASH_X_ARGS = ' '

                    ADDITIONAL_ARGS = 'integrationTest'
                }
            }
        }
        stage('Compile / Test') {
            agent {
                dockerfile {
                    filename 'Dockerfile.JenkinsBuildEnv'
                    reuseNode true
                }
            }
            when {
                not { expression { return params.QA_PROMOTION } }
            }
            steps {
                  echo "hello"
            }
        }
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
        stage('Publish Release RPMs') {
            when {
                branch 'release/*'
                not { changeRequest() }
                not { expression { return params.QA_PROMOTION } }
            }
            agent {
                dockerfile {
                    filename 'Dockerfile.JenkinsBuildEnv'
                    reuseNode true
                }
            }
            steps {
                echo "hello"
            }
        }
        stage('Build Docker Images') {
            when {
                branch 'release/*'
                not { changeRequest() }
                not { expression { return params.QA_PROMOTION } }
            }
            steps {
                echo "hello"
            }
        }
        stage('QA Promotion') {
            when {
                expression { return params.QA_PROMOTION }
            }
            agent {
                dockerfile {
                    filename 'Dockerfile.JenkinsBuildEnv'
                    reuseNode true
                }
            }
            steps {
                echo "hello"
            }
        }
    }
}
