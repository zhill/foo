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
    tag = repository + ":" + "auto_${env.BUILD_ID}"
    // Build the image and push it to a staging repository
    docker.withRegistry("http://localhost:5001") {
      app = docker.build(tag)
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
      // This is a different url from the push because anchore-engine doesn't use the mounted docker.sock so it needs the actual container name if using docker-compose to run them all.
      def imagesLine = "registry:5000/" + tag + " " + dockerfile
      writeFile file: anchorefile, text: imagesLine
      anchore name: anchorefile, bailOnFail: false
    }
  }

  stage('Cleanup') {
    // Delete the docker image and clean up any allotted resources
    sh'''
      docker rmi $(docker images --filter "label=src=jenkins_demo" -q)
    '''
    //  for i in `cat anchore_images | grep foo | awk '{print $1}'`;do docker rmi $i; done
    //'''
  }
}
