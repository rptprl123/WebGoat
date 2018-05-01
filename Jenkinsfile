pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    mvn -B install -Dmaven.test.skip=true
                '''
      }
    }
    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            nexusPolicyEvaluation(iqApplication: 'webgoat8', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
            sh '''cd webgoat-server
docker build -t webgoat/webgoat-8.0 .
                    '''
          }
        }
      }
    }
    stage('Test Container') {
      parallel {
        stage('Test Container') {
          steps {
            echo '...run container and test it'
          }
          post {
            success {
              echo '...the Test Scan Passed!'

            }

            failure {
              echo '...the Test  FAILED'
              error '...the Container Test FAILED'

            }

          }
        }
        stage('error') {
          steps {
            sh 'docker save webgoat/webgoat-8.0 -o $WORKSPACE/webgoat.tar'
            nexusPolicyEvaluation(iqStage: 'stage', iqApplication: 'webgaot8')
          }
        }
      }
    }
    stage('Publish Container') {
      when {
        branch 'master'
      }
      steps {
        sh '''
                    docker tag webgoat/webgoat-8.0 mycompany.com:5000/webgoat/webgoat-8.0:8.0
                    docker push mycompany.com:5000/webgoat/webgoat-8.0
                '''
      }
    }
  }
  tools {
    maven 'M3'
  }
}