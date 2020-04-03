pipeline {
  agent any
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
  }
  post {
    always {
        deleteDir() /* clean up our workspace */
    }
  }
}