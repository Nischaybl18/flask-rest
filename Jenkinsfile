node {
    
  stage 'Checkout'

  git 'https://github.com/Nischaybl18/flask-rest.git'
        
  stage 'Package Docker image'

  def img = docker.build('flask-app:latest', '.')

  stage 'Publish'
  docker.withRegistry('https://hub.docker.com', 'nischaybl18') {
     img.push('latest')
  }

}
