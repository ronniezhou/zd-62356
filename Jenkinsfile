#!/usr/bin/env groovy

pipeline {
  agent none
  stages {
    stage('Main') {
      parallel {
        stage('Windows') {
          agent any
          stages {
            stage('Windows build') {
              steps {
                println currentBuild.getBuildCauses()
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
          agent any
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
          agent any
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
