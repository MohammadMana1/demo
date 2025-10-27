pipeline {
  agent any

  tools {
    jdk   'jdk-17'
    maven 'maven-3.9.x'
  }

  triggers {
    pollSCM('H/2 * * * *')
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    WORKDIR = ''
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Show workspace') {
      steps {
        sh '''
          echo "PWD:"; pwd
          echo "List (top level):"
          ls -la
          echo "--- Searching for pom.xml (depth 2) ---"
          find . -maxdepth 2 -name pom.xml -print
        '''
      }
    }

    stage('Detect Maven project folder') {
      steps {
        script {
          def out = sh(returnStdout: true, script: '''
            set -e
            f=$(find . -maxdepth 2 -name pom.xml | head -n 1)
            if [ -z "$f" ]; then
              exit 2
            fi
            d=$(dirname "$f")
            d=${d#./}
            printf "%s" "$d"
          ''').trim()
          if (!out) {
            error "No pom.xml found (depth ≤ 2). Commit your Spring Boot project."
          }
          env.WORKDIR = (out == '.' ? '' : out)
          echo "Detected WORKDIR='${env.WORKDIR}' ('' means repo root)"
        }
      }
    }

    stage('Build') {
      steps {
        script {
          if (env.WORKDIR) {
            dir(env.WORKDIR) { sh 'mvn -B -DskipTests clean package' }
          } else {
            sh 'mvn -B -DskipTests clean package'
          }
        }
      }
    }

    stage('Repackage Fat JAR') {
      steps {
        script {
          if (env.WORKDIR) {
            dir(env.WORKDIR) { sh 'mvn -B spring-boot:repackage' }
          } else {
            sh 'mvn -B spring-boot:repackage'
          }
        }
      }
    }

    stage('Archive Artifact') {
      steps {
        script {
          def jarGlob = env.WORKDIR ? "${env.WORKDIR}/target/*.jar" : "target/*.jar"
          archiveArtifacts artifacts: jarGlob, fingerprint: true
        }
      }
    }

    stage('Unit Tests') {
      steps {
        script {
          if (env.WORKDIR) {
            dir(env.WORKDIR) { sh 'mvn -B test' }
          } else {
            sh 'mvn -B test'
          }
        }
      }
      post {
        always {
          script {
            def reports = env.WORKDIR ? "${env.WORKDIR}/target/surefire-reports/*.xml" : "target/surefire-reports/*.xml"
            junit allowEmptyResults: true, testResults: reports
          }
        }
      }
    }
  }

  post {
    success { echo " Build finished successfully." }
    failure { echo " Build failed. Check the red stage’s Console Output." }
  }
}
