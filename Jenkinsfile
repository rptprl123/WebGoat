pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    mvn -B  install -DskipTests'''
      }
    }
    stage('Build App - Scan') {
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
      }
    }
         stage('Build Container') {
          steps {
            sh '''#!/bin/bash
cd webgoat-server
whoami
pwd
echo $PATH
docker login 192.168.0.231 -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
echo "Login"
docker build --no-cache -t webgoat/webgoat-8.0:latest .
echo "build"
docker images
'''
          }
        }

        stage('Docker Container Scan') {
          steps {
            sh 'docker save webgoat/webgoat-8.0 -o $WORKSPACE/webgoat.tar'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'webgoat8', iqScanPatterns: [[scanPattern: 'webgoat.tar']])
      }
    }
    stage('Container Harbor Push') {
      steps {
        sh '''
                    docker tag webgoat/webgoat-8.0:latest 192.168.0.231/webgoat/webgoat-8.0:latest
                    docker push 192.168.0.231/webgoat/webgoat-8.0:latest
                '''
      }
    }
  }
  tools {
    maven 'M3'
    jdk 'JDK8'
  }
}
