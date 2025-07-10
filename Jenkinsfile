pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven 3'  // Set this to your configured Maven tool name in Jenkins
        JAVA_HOME = tool 'JDK 17'    // Adjust based on your JDK configuration
        ARTIFACTORY_URL = "http://localhost:5040/artifactory""
        APP_NAME = "myapp"
        VERSION = "1.0"
        WAR_FILE = "${APP_NAME}-${VERSION}.war"
        CONTAINER_NAME = "tomcat-server"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install verify"
            }
        }

        stage('Publish to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CRED', usernameVariable: 'manish', passwordVariable: 'manishraj')]) {
                    sh '''
                    curl -u $ART_USER:$ART_PASS -T target/$WAR_FILE \
                      "$ARTIFACTORY_URL/$APP_NAME/$VERSION/$WAR_FILE"
                    '''
                }
            }
        }

        stage('Download from Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CRED', usernameVariable: 'manish', passwordVariable: 'manishraj')]) {
                    sh '''
                    curl -u $ART_USER:$ART_PASS -O "$ARTIFACTORY_URL/$APP_NAME/$VERSION/$WAR_FILE"
                    '''
                }
            }
        }

        stage('Deploy to Dockerized Tomcat') {
            steps {
                sh '''
                docker cp $WAR_FILE $CONTAINER_NAME:/usr/local/tomcat/webapps/$APP_NAME.war
                docker restart $CONTAINER_NAME
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                sleep 10
                curl -I http://localhost:8081/$APP_NAME/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
// Note: Ensure that the Jenkins server has access to Docker and the necessary permissions to run Docker commands.
// Also, ensure that the Artifactory credentials are set up in Jenkins with the ID 'ARTIFACTORY_CRED'.
// Adjust the WAR file path and container name as per your setup.  
// The Tomcat server should be running in a Docker container named 'tomcat-server'.
// The script assumes that the Tomcat server is accessible at http://localhost:8081.