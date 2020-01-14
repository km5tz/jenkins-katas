pipeline {
  agent any
  stages {
    stage('clone down') {
      steps {
        stash(excludes: '.git', name: 'code')
        deleteDir()
      }
    }

    stage('Test and build') {
      parallel {
        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
            stash(excludes: '.git', name: 'code')
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash(excludes: '.git', name: 'code')
          }
        }

      }
    }

    stage('build docker') {
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
      }
    }

    stage('push docker') {
      when {
        branch 'master'
      }
      environment {
        DOCKERCREDS = credentials('docker_login')
      }
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

    stage('component test') {
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }

  }
  environment {
    docker_username = 'km5tz'
  }
}