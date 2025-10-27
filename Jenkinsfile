pipeline {
  agent any

  tools {
    jdk 'jdk-17'
    maven 'maven-3.9.x'
  }

  triggers {
    pollSCM('H/2 * * * *')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build') {
      steps { sh 'mvn -B -DskipTests clean package' }
    }
    stage('Repackage Fat JAR') {
      steps { sh 'mvn -B spring-boot:repackage' }
    }
    stage('Archive Artifact') {
      steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
    }
    stage('Unit Tests') {
      steps { sh 'mvn -B test' }
      post { always { junit 'target/surefire-reports/*.xml' } }
    }
  }
}
