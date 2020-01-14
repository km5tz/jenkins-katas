pipeline {
  agent any
  stages {
    stage('clone down') {
      agent {
        label 'host'
      }
      steps {
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Say Hello') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
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
          }
        }

      }
    }

    stage('docker_push') {
      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

  }
  environment {
    docker_username = 'km5tz'
    DOCKERCREDS = credentials('docker_login')
  }
}