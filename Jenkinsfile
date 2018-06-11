node {
    def app

    stage('Build image') {
        /* This builds the actual image; synonymous to docker build on the command line */

        app = docker.build('wazowskis/foo:${env.BUILD_NUMBER}')
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image. */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags*/
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push()
            app.push('latest')
        }
    }
}
