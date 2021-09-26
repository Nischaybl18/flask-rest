
pipeline{

environment {
    GIT_COMMIT_SHORT_HASH = sh (script: "git rev-parse --short HEAD", returnStdout: true)
}

agent any

stages{
    stage('Init'){
        steps{
            //checkout scm;
        script{
        env.BASE_DIR = pwd()
        env.CURRENT_BRANCH = env.master
        env.IMAGE_TAG = getImageTag(env.master)
        env.TIMESTAMP = getTimeStamp();
        env.APP_NAME= getEnvVar('APP')
        env.IMAGE_NAME = getEnvVar('flaskapp')
        env.PROJECT_NAME=getEnvVar('App-Test')
        env.DOCKER_REGISTRY_URL=getEnvVar('https://hub.docker.com/repository/docker/nischaybl18/flaskapp')
        env.RELEASE_TAG = getEnvVar('App-Release')
        env.DOCKER_PROJECT_NAMESPACE = getEnvVar('flaskapp')
        env.DOCKER_IMAGE_TAG= "${https://hub.docker.com/repository/docker/nischaybl18/flaskapp}/${flaskapp}/${APP}:${App-Release}"
        env.JENKINS_DOCKER_CREDENTIALS_ID = getEnvVar('')        
        env.JENKINS_GCLOUD_CRED_ID = getEnvVar('JENKINS_GCLOUD_CRED_ID')
        env.GCLOUD_PROJECT_ID = getEnvVar('GCLOUD_PROJECT_ID')
        env.GCLOUD_K8S_CLUSTER_NAME = getEnvVar('GCLOUD_K8S_CLUSTER_NAME')
        env.JENKINS_GCLOUD_CRED_LOCATION = getEnvVar('JENKINS_GCLOUD_CRED_LOCATION')

        }

        }
    }

    stage('Cleanup'){
        steps{
            sh '''
            docker rmi $(docker images -f 'dangling=true' -q) || true
            docker rmi $(docker images | sed 1,2d | awk '{print $3}') || true
            '''
        }

    }
    stage('Build'){
        steps{
            withEnv(["APP_NAME=${APP_NAME}", "PROJECT_NAME=${PROJECT_NAME}"]){
                sh '''
                docker build -t ${DOCKER_REGISTRY_URL}/${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG} --build-arg APP_NAME=${IMAGE_NAME}  -f app/Dockerfile app/.
                '''
            }   
        }
    }
    stage('Publish'){
        steps{
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${JENKINS_DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWD']])
            {
            sh '''
            echo $DOCKER_PASSWD | docker login --username ${DOCKER_USERNAME} --password-stdin ${DOCKER_REGISTRY_URL} 
            docker push ${DOCKER_REGISTRY_URL}/${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG}
            docker logout
            '''
            }
        }
    }
    stage('Deploy'){
        steps{
        withCredentials([file(credentialsId: "${JENKINS_GCLOUD_CRED_ID}", variable: 'JENKINSGCLOUDCREDENTIAL')])
        {
        sh """
            #!/bin/sh -xe
            gcloud auth activate-service-account --key-file=${JENKINSGCLOUDCREDENTIAL}
            gcloud config set compute/zone asia-southeast1-a
            gcloud config set compute/region asia-southeast1
            gcloud config set project ${GCLOUD_PROJECT_ID}
            gcloud container clusters get-credentials ${GCLOUD_K8S_CLUSTER_NAME}
            
            chmod +x $BASE_DIR/k8s/process_files.sh
            cd $BASE_DIR/k8s/
            ./process_files.sh $GCLOUD_PROJECT_ID ${IMAGE_NAME} "${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG}" "./${IMAGE_NAME}/" $TIMESTAMP
            cd $BASE_DIR/k8s/$IMAGE_NAME/.
            kubectl apply --force=true --all=true --record=true -f $BASE_DIR/k8s/$IMAGE_NAME/
            kubectl rollout status --watch=true --v=8 -f $BASE_DIR/k8s/$IMAGE_NAME/$IMAGE_NAME-deployment.yml
            gcloud auth revoke --all
            """
        }
        }
    }
}
