pipeline {
    agent any

    environment {
        SONAR_VERSION = '8.9.1'
        SONAR_HOST_URL = 'http://13.71.99.161:9090'
        SONAR_LOGIN = credentials('sonarqube-token')
	NEXUS_URL = 'http://159.223.191.140:8081'
        NEXUS_CREDENTIALS_ID = 'nexus-cred'
        PACKAGE_VERSION = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code.
                git branch: 'main', url: 'https://github.com/rupjyot13/Jenkin-maven-build-pipeline.git'
            }
        }

        stage('Build') {
            steps {
                // Buildin project with Maven
                sh 'mvn clean package'
            }
        }
		
	 stage('Unit Test') {
            steps {
                // Run unit tests with JUnit to generate test reports
                sh 'mvn test'
            }

            post {
                always {
                    // Archive JUnit test results
                    junit '**/target/tests-reports/*.xml'
                }
            }
        }

         stage('Static Code Analysis') {
            steps {
                // Execute SonarQube analysis
                sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_LOGIN} -Dsonar.projectKey=test_project -Dsonar.projectName=test_project -Dsonar.projectVersion=1.0"
            }
        }
		
	    stage('Publish to Nexus') {
              steps {
                script {
                    // Deploy package to Nexus
                    def nexusArtifactUploader = NexusArtifactUploader.fromMaven(projectVersion: PACKAGE_VERSION)
                    nexusArtifactUploader.deployArtifacts(
                        artifacts: [
                            [artifactId: 'sprinboot', groupId: 'com.example', version: PACKAGE_VERSION, classifier: '', file: 'sprinboot.jar']
                        ],
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '${NEXUS_URL}',
                        credentialsId: '${NEXUS_CREDENTIALS_ID}'
                    )
                }
            }
        }
		
	    stage('Download from Nexus') {
              steps {
                // Download the artifact from Nexus
                script {
                    def nexusArtifactDownloader = NexusArtifactDownloader.fromMaven()
                    nexusArtifactDownloader.downloadArtifact(
                        groupId: 'com.example',
                        artifactId: 'sprinboot',
                        version: PACKAGE_VERSION,
                        classifier: '',
                        packaging: 'jar',
                        nexusUrl: '${NEXUS_URL}',
                        credentialsId: '${NEXUS_CREDENTIALS_ID}'
                    )
                }
            }
        }
		
	   stage('Deploy to QA Environment') {
             agent { label 'QA environment' }
             steps {
                sh 'java -jar sprinboot.jar'
            }
        }
    }

    post {
        success {
            // Enforce quality gate before proceeding to the next environment
            script {
                def qualityGateStatus = waitForQualityGate()
                if (qualityGateStatus != 'OK') {
                    error "Quality gate failed: ${qualityGateStatus}"
                }
            }
        }
    failure {
            // Send email notification in case of build failure
            emailext (
                to: 'rupeshshinde1@gmail.com', 
                subject: 'Build Failed message', 
                body: 'The build failed. Please check the Jenkins console output for details.',
                attachLog: true
            )
        }
    }
}

def waitForQualityGate() {
    def qg = waitForQualityGate() 
    return qg.getStatus()
}
