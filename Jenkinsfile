pipeline {
  agent any
  environment {
    docker_username = 'anttirastas'
  }
  stages {
    stage('clone down') {
        agent {
          label 'host'
        }
        steps {
          stash excludes: '.git', name: 'code'
        }
    }
    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }
        stage('build app') {
          options {
            skipDefaultCheckout()
          }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            // first list the contents of the workspace, 
            sh 'ls'
            stash includes: 'app/build/libs/', name: 'binaries'
            // then use the deleteDir() keyword to delete the workspace,
            deleteDir()
            // and finally list the content again after the deletion, to verify that they were deleted. 
            sh 'ls'
          }
        }
        stage('test app') {
          options {
            skipDefaultCheckout()
          }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }
    stage('push to Docker Hub') {
      environment {
        DOCKERCREDS = credentials('docker_hub') //use the credentials just created in this stage
      }
      steps {
        input message: 'Are you sure you want to push to Docker Hub?', ok: 'Absotelutely!'
        unstash 'binaries'
        unstash 'code'
        sh 'ls'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }
  }
  post {
    always {
      deleteDir() /* clean up our workspace */
    }
  }
}