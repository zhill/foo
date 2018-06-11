node {
  def app

  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    /* This builds the actual image; synonymous to docker build on the command line */

    app = docker.build('wazowskis/foo:${BUILD_NUMBER}')
  }

  stage('Test') {
    /* Ideally, we would run a test framework against our image. */

    app.inside {
        sh 'echo "Tests passed"'
    }
  }

  stage('Push') {
    /* Finally, we'll push the image with two tags*/
    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-wazowskis') {
        app.push()
    }
  }
}
