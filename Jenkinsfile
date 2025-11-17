pipeline {
    agent any

    environment {
        NEXUS_URL = "http://localhost:8081"
        NEXUS_REPO = "maven-releases"
        GROUP_ID = "com.example"
        ARTIFACT_ID = "demo"
        VERSION = "1.0.0"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Jar') {
            steps {
                dir('demo') {
                    sh 'mvn -B clean package'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'demo/target/*.jar', fingerprint: true
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-admin',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        JAR_FILE=demo/target/demo-0.0.1-SNAPSHOT.jar

                        GROUP_PATH=`echo "$GROUP_ID" | tr '.' '/'`

                        echo "Uploading $JAR_FILE ..."
                        echo "Group path: $GROUP_PATH"

                        curl -v -u "$NEXUS_USER:$NEXUS_PASS" \
                          --upload-file "$JAR_FILE" \
                          "$NEXUS_URL/repository/$NEXUS_REPO/$GROUP_PATH/$ARTIFACT_ID/$VERSION/$ARTIFACT_ID-$VERSION.jar"
                    '''
                }
            }
        }
    }
}
