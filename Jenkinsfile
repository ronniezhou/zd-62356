#!groovy

def gradleWithRemoteCache(String tasks, String extraArgs='') {
  withCredentials([
    usernamePassword(credentialsId: 'docker-deploy',
    usernameVariable: 'user',
    passwordVariable: 'password')
  ]) {
    sh """./source/gradlew $tasks -i \
      -Pgradle.cache.push=true \
      -PartifactoryUser=$user \
      -PartifactoryPassword=$password \
      $extraArgs
    """
  }
}

pipeline {

    agent {
        label 'docker-swarm'
    }

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
            when {
                anyOf { branch 'master' ; branch 'release/*' }
                not { changeRequest() }
                not { expression { return params.QA_PROMOTION } }
                not { expression { return params.EMERGENCY } }
            }
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
                // VERSION and THE_RELEASE are required here since some components use the gradle application
                // plugin which creates a tarball whose name includes these tags. The tarballs (here) and the
                // corresponding rpms (below) must have matching tags.
                /*gradleWithRemoteCache """-p source clean build tools:batchutil:archtest platform:ctld-batch:unusedobjectcleanup:integrationTest \
                    platform:ctld-batch:unusedobjectcleanup:jacocoTestCoverageVerification \
                    -Pversion=${VERSION} -Prelease=${THE_RELEASE} -PshowViolations=false --refresh-dependencies --parallel \
                    ${DASH_X_ARGS} ${ADDITIONAL_ARGS}""" */
                gradleWithRemoteCache """-p source clean build \
                    -Pversion=${VERSION} -Prelease=${THE_RELEASE} -PshowViolations=false --refresh-dependencies --parallel \
                    ${DASH_X_ARGS} ${ADDITIONAL_ARGS}"""
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
                // compute deltas from INITIAL_COMMIT and build only those rpms. then publish them to artifactory
                gradleWithRemoteCache """-p source buildChangedRpms publishChangedRpms \
                    -PpreviousCommit=${INITIAL_COMMIT} \
                    -Pversion=${VERSION} -Prelease=${THE_RELEASE} \
                    -PpublishUser=$ARTIFACTORY_CREDS_USR -PpublishPassword=$ARTIFACTORY_CREDS_PSW"""
                archiveArtifacts artifacts: '**/build/distributions/*.rpm', allowEmptyArchive: true
            }
        }
        stage('Build Docker Images') {
            when {
                branch 'release/*'
                not { changeRequest() }
                not { expression { return params.QA_PROMOTION } }
            }
            steps {
                withDockerRegistry([url: "https://docker.vrsn.com", credentialsId: "docker-deploy"]) {
                    // copy docker folder to batches that don't have one, then build those images
                    sh "./populate-batch-docker-dirs.sh -b ${INITIAL_COMMIT}"
                    sh "./process-docker-images.sh -b ${INITIAL_COMMIT} -- -v ${VERSION} -r ${THE_RELEASE} -p"
                    gradleWithRemoteCache """-p source buildChangedDockerImages -PpushDockerImage=true \
                        -PpreviousCommit=${INITIAL_COMMIT} \
                        -Pversion=${VERSION} -Prelease=${THE_RELEASE}"""
                    archiveArtifacts artifacts: '**/build/*.docker', allowEmptyArchive: true
                }
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
                gradleWithRemoteCache """-p source promoteChangedRpms \
                    -PpreviousCommit=${INITIAL_COMMIT} -Pversion=${VERSION} -Prelease=${THE_RELEASE} \
                    -PpublishUser=$ARTIFACTORY_CREDS_USR -PpublishPassword=$ARTIFACTORY_CREDS_PSW"""
            }
        }
    }
}
