pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "spottabathula69/cicd-kube-docker"
        registryCredentials = "dockerhub"
    }

    stages{

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
/*
        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
*/	
	stage("Buid App Image") {
		steps {
			script {
                echo " Start build docker image"
				//dockerImage = docker.build.registry + ":V$BUILD_NUMBER"
                def dockerImage = "${docker.build.registry}:V${BUILD_NUMBER}"
                echo "Building Docker image: ${dockerImage}"
                docker.build(dockerImage, '.')

			}
		}
	}
	stage("Upload Image") {
		steps {
			script {
				docker.withRegistry('', registryCredentials) {
                    echo "Pushing Docker image:"
					dockerImage.push("V$BUILD_NUMBER")
					dockerImage.push("latest")
				}
			}
		}
	}
        stage("Remove Unused docker Image") {
                steps {
			        sh "docker rmi $registry:V$BUILD_NUMBER"
                }
        }

        stage("Kubernetes Deploy") {
	   	agent { label "KOPS" }
                steps {
                        sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
                }
        }




    }


}

