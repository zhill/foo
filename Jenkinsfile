node {
  def app
  def dockerfile
  def anchorefile
  def repository
  def tag

  stage('Checkout') {
    // Clone the git repository
    checkout scm
    def path = sh returnStdout: true, script: "pwd"
    path = path.trim()
    dockerfile = path + "/Dockerfile"
    anchorefile = path + "/anchore_images"
  }

  stage('Build') {
    repository = "test_images/foo"
    tag = ${BUILD_NUMBER}
    // Build the image and push it to a staging repository
    docker.withRegistry("https://registry:5001/v2/") {
      app = docker.build("${repository}:${tag}")
      app.push()
    }
  }

  stage('Parallel Test and Analyze') {
    parallel test: {
      app.inside {
          sh 'echo "Tests passed"'
      }
    },
    analyze: {
      def imagesLine = "registry:5001/${repository}:${tag} " + dockerfile
      writeFile file: anchorefile, text: imagesLine
      anchore name: anchorefile, bailOnFail: false
    }
  }

  stage('Cleanup') {
    // Delete the docker image and clean up any allotted resources
    sh'''
      for i in `cat anchore_images | grep foo | awk '{print $1}'`;do docker rmi $i; done
    '''
  }
}
