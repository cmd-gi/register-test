pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'java17'
        maven 'maven3'
    }

	environment {
	    APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "cmdgi"
        DOCKER_PASS = credentials('dockerhub')
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/cmd-gi/register-test', 
                    credentialsId: 'github'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis"){
           steps {
	           script {
		            withSonarQubeEnv('sonarqube-server') { 
                        sh "mvn sonar:sonar"
		            }
	           }	
           }
       }

	   stage("Quality Gate"){
		   steps {
			   script {
					waitForQualityGate abortPipeline: false
				}	
			}
	   }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }

		stage("Trivy Scan") {
           steps {
               script {
	               sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image cmdgi/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
    }
}
