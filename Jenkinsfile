pipeline {
    agent { label 'Jenkins-Agent' }
	 triggers {
        //pollSCM('* * * * *')
		
		 // Active le déclenchement via webhook GitHub
		 githubPush()
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
	environment {
	    	APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "amidou666"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
			JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }
	
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/ouattaraa/register-app-youtube-devops-course'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
		stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }


		stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }

		/* stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image amidou666/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }*/


		stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }


		stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-48-44-133.eu-north-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }


		
    }

	   post {
    always {
        script {
            def buildStatus = currentBuild.currentResult
            def color = (buildStatus == 'SUCCESS') ? 'green' : (buildStatus == 'FAILURE' ? 'red' : 'orange')

            emailext(
                subject: "📌 [${env.JOB_NAME}] Build #${env.BUILD_NUMBER} - ${buildStatus}",
                body: """
                    <html>
                        <body style="font-family: Arial, sans-serif; color:#333;">
                            <h2 style="color:${color};">🔔 Build Notification</h2>
                            <p>
                                <b>Job:</b> ${env.JOB_NAME}<br>
                                <b>Build Number:</b> #${env.BUILD_NUMBER}<br>
                                <b>Status:</b> <span style="color:${color}; font-weight:bold;">${buildStatus}</span><br>
                                <b>Duration:</b> ${currentBuild.durationString}
                            </p>
                            <p>
                                👉 <a href="${env.BUILD_URL}" style="color:#1a73e8;">Cliquez ici pour voir les logs Jenkins</a>
                            </p>
                            <hr>
                            <p style="font-size:12px; color:#777;">
                                🚀 Jenkins Pipeline Notification - Envoyé automatiquement par Jenkins CI/CD
                            </p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html',
                to: "amidouattara51@gmail.com"
            )
        }
    }
}

	
}
