pipeline {
  agent {
    label 'jdk8'
  }
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
            sh '''
                        cd webgoat-server
                        docker build -t webgoat/webgoat-8.0 .
                    '''
          }
        }
      }
    }
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
    stage('Scan Container') {
      steps {
        sh "docker save webgoat/webgoat-8.0 -o ${env.WORKSPACE}/webgoat.tar"
        nexusPolicyEvaluation(iqApplication: 'webgoat8', iqStage: 'stage', iqScanPatterns: [[scanPattern: '*.tar']])
      }
      post {
        success {
          echo '...the IQ Scan PASSED'
          postGitHub(commitId, 'success', 'analysis', 'Nexus Lifecycle Container Analysis succeeded', "${policyEvaluationResult.applicationCompositionReportUrl}")

        }

        failure {
          echo '...the IQ Scan FAILED'
          postGitHub(commitId, 'failure', 'analysis', 'Nexus Lifecycle Containe Analysis failed', "${policyEvaluationResult.applicationCompositionReportUrl}")
          error '...the IQ Scan FAILED'

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