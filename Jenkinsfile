pipeline {
  agent any

  tools {
    jdk   'jdk-17'
    maven 'maven-3.9.x'
  }

  triggers { pollSCM('H/2 * * * *') }

  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Show workspace') {
      steps {
        sh '''
          echo "PWD:"; pwd
          echo "Top-level files:"
          ls -la
          echo "--- pom.xml search ---"
          find . -maxdepth 2 -name pom.xml -print
        '''
      }
    }

    stage('Build') {
      steps {
        dir('demo') {
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }

    stage('Repackage Fat JAR') {
      steps {
        dir('demo') {
          sh 'mvn -B spring-boot:repackage'
        }
      }
    }

    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: 'demo/target/*.jar', fingerprint: true
      }
    }

    stage('Unit Tests') {
      steps {
        dir('demo') {
          sh 'mvn -B test'
        }
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'demo/target/surefire-reports/*.xml'
        }
      }
    }
  }

  post {
    success { echo '✅ Build finished successfully.' }
    failure { echo '❌ Build failed. Check the red stage’s Console Output.' }
  }
}
