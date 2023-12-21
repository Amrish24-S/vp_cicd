pipeline {
    agent any
    environment {
        registry = "amrish24/vproapp"
        registryCredential = 'dockerhub'
    }
    stages {
        stage ('BUILD') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post{
                success{
                    echo "====++++ Now Archiving ++++===="
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }
        stage ('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('INTEGRATED TESTING') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('CHECKSTYLE ANALYSIS') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success{
                    echo "Generated Code Analysis Result"
                }
            }
        }
        stage ('SONARQUBE ANALYSIS') {
            environment {
                scannerHome = tool 'Sonar4.7'
            }
            steps {
                withSonarQubeEnv('Sonar-server') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
                timeout (time: 10, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('BUILD IMAGE') {
            steps {
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }
        stage ('PUSH IMAGE') {
            steps {
                script {
                    docker.withRegistry('', registryCredential){
                        dockerImage.push ("V$BUILD_NUMBER")
                        dockerImage.push ("latest")
                    }
                }
            }
        }
        stage ('REMOVE UNUSED IMAGE') {
            steps {
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }
    }
}
