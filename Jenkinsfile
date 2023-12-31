pipeline {
	agent {
	   	label 'silver-node'
	}
	tools {
		maven "MAVEN3"
		jdk "OracleJDK11"
	}

	environment {
		registryCredential = 'ecr:us-east-1:awscreds'
		appRegistry = "307416798703.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
		vprofileRegistry = "https://307416798703.dkr.ecr.us-east-1.amazonaws.com"
		cluster = "vprofile"
		service = "vprofileappsvc"
	}

	stages {
		stage ('Fetch Code') {
			steps {
				git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
			}
		}
		stage ('Test') {
			steps {
				sh 'mvn test'
			}
		}
		stage ('CODE ANALYSIS WITH CHECKSTYLE') {
			steps {
				sh 'mvn checkstyle:checkstyle'
			}
			post {
				success {
					echo 'Generated Analysis Result'
				}
			}
		}

		stage ('build && SonarQube Analysis') {
			environment {
				scannerHome = tool 'sonar4.7'
			}
            steps {
              withSonarQubeEnv('sonar') {
               sh '''${scannerHome}/bin/sonar-scanner -X -Dsonar.projectKey=vprofile \
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
       stage('Build App Image') {
       		steps {
       
         		script {
                	dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             	}

        	}
		}
		stage('Upload App Image') {
            steps{
          	   script {
               		docker.withRegistry( vprofileRegistry, registryCredential ) {
                	dockerImage.push("$BUILD_NUMBER")
                	dockerImage.push('latest')
                    }
            	}
          	}
        }
	}	
}
