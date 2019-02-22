pipeline {
    agent any
    
    parameters { 
         string(name: 'tomcat_dev', defaultValue: '35.166.210.154', description: 'Staging Server')
         string(name: 'tomcat_prod', defaultValue: '34.209.233.6', description: 'Production Server')
    } 

    triggers {
         pollSCM('* * * * *') // Polling Source Control
     }

    tools { 
        maven 'localMaven' 
    //    jdk 'jdk8' 
    }

    stages{
        stage('Build'){
            steps {
	        sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage ('Deployments'){
            parallel{
                stage ('Deploy to Staging'){
                    steps {
                        //sh "scp -i /home/jenkins/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat7/webapps"
			echo "Deploying to Staging at ${params.tomcat_dev}"
			build job: 'deploy-to-staging'
                    }
                }

                stage ("Deploy to Production"){
                    steps {
			timeout(time:5, unit:'DAYS') {
				input message: 'Approve PRODUCTION deployment?'
			}

                        //sh "scp -i /home/jenkins/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_prod}:/var/lib/tomcat7/webapps"
			echo "Deploying to Production at ${params.tomcat_prod}"
			build job: 'deploy-to-prod'
                    }
		    post {
			success {
				echo 'Code deployed to Production'
			}
			failure {
				echo 'Deployment failed'
			}
		    }
                }
            }
        }
    }
}
