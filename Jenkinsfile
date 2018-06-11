node {
  def app

  stage('Checkout') {
    // Clone the git repository
    checkout scm
  }

  stage('Build') {
    // Build the image and push it to a staging repository
    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-wazowskis') {
      app = docker.build('wazowskis/foo:${BUILD_NUMBER}')
      app.push()
    }
  }

  stage('Test') {
    // Run tests. Ideally, we would run a test framework against our image
    app.inside {
        sh 'echo "Tests passed"'
    }
  }

  stage('Analyze') {
    // Submit the image to anchore plugin for analysis
    sh 'echo "docker.io/wazowskis/foo:${BUILD_NUMBER} `pwd`/Dockerfile" > anchore_images'
    anchore name: 'anchore_images', bailOnFail: false
  }

  stage('Cleanup') {
    // Delete the docker image and clean up any allotted resources
    sh'''
      for i in `cat anchore_images | grep foo | awk '{print $1}'`;do docker rmi $i; done
    '''
  }
}
