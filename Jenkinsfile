#!/usr/bin/env groovy

pipeline {
  agent none
  stages {
    stage('Main') {
      parallel {
        stage('Windows') {
          agent {
            label 'win'
          }
          stages {
            stage('Windows build') {
              steps {
                echo 'Windows building...'
              }
            }
            stage('Windows unit tests') {
              steps {
                echo 'Windows unit testing...'
              }
            }
            stage('Windows deploy') {
              steps {
                echo 'Windows deploying...'
              }
            }
          } // end stages
        } // end stage windows

        stage('Linux') {
          agent {
            label 'linux'
          }
          stages {
            stage('Linux build') {
              steps {
                echo 'Linux building...'
              }
            }
            stage('Linux unit test') {
              steps {
                echo 'Linux unit testing...'
              }
            }
            stage('Linux deploy') {
              steps {
                echo 'Linux deploying...'
              }
            }
          } // end stages
        } // end stage linux

        stage('Mac') {
          agent {
            label 'mac'
          }
          stages {
            stage('Mac build') {
              steps {
                echo 'Mac building...'
              }
            }
            stage('Mac unit test') {
              steps {
                echo 'Mac unit testing...'
              }
            }
            stage('Mac deploy') {
              steps {
                echo 'Mac deploying...'
              }
            }
          } // end stages
        } // end stage Mac
      } // end parallel
    } // end stage Main
  } // end stages
} // end pipeline
