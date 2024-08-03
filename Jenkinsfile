pipeline {

    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Target Branch')
    }

/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "nelsonosagie/nelapp"
        registryCredential = 'docker-cred' /*create this cred*/
    }

    stages{

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository'){
             steps{
             git branch: 'main', url: 'https://github.com/Nelgit007/k8s-app-deployment.git'
            }
        }

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        // stage('CODE ANALYSIS with SONARQUBE') {

        //     environment {
        //         scannerHome = tool 'mysonarscanner4'
        //     }

        //     steps {
        //         withSonarQubeEnv('sonar-scan') {
        //             sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //            -Dsonar.projectName=vprofile-repo \
        //            -Dsonar.projectVersion=1.0 \
        //            -Dsonar.sources=src/ \
        //            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //            -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        //         }

        //         timeout(time: 10, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        stage('Build App Image') {
          steps {
            script {
              dockerImage = docker.build registry + ":V$BUILD_NUMBER"
            }
          }
        }

        stage('Upload Image'){
          steps{
            script {
              docker.withRegistry('', registryCredential) {
                dockerImage.push("V$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:V$BUILD_NUMBER"
            sh "docker rmi $registry:latest"
          }
        }

        stage('Kubernetes Deploy') {
          agent {label 'KOPS'}
            steps {
              sh "helm upgrade --install --force vpro-stack helm/vprocharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace production"
            }
        }
    }


}
