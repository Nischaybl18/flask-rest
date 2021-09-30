pipeline{
    agent any

    environment{
        dockerImage = ''
        registry = 'nischaybl18/flaskapp'
        registryCredential = 'nischaybl18-dockerhub'
    }
    stages{
        stage('GIT Clone'){
            steps{
                git credentialsId: 'nischaybl18-github', url: 'https://github.com/Nischaybl18/flask-rest.git'
            }
		}
        stage('Build'){
            steps{
                script{
                    dockerImage = docker.build registry
                }
            }
		}
         stage('Publish'){
            steps{
                script{
                      docker.withRegistry( '', registryCredential ) {
                      dockerImage.push()
                      }
                }
            }

        }
        stage('Deploy'){
            steps{
            script{
                sshagent(['k8s-ma']) {
    			sh "scp -o StrictHostKeyChecking=no services.yaml pods.yaml ingress.yaml ubuntu@ip-172-31-29-251:/home/ubuntu/"
                    	script{
                        	try{
                            		sh "ssh ubuntu@ip-172-31-29-251 kubectl apply -f ."
                        	}catch(error){
                            		sh "ssh ubuntu@ip-172-31-29-251 kubectl create -f ."
                        	}
                    	}
               	 }
        }
/*
                configs: '', kubeConfig: [path: ''], kubeconfigId: 'k8s', secretName: '', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']
 */

                 }
        }

   }
}
