node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("zumar201/cw2")
    }

    stage('Test image') {
        /* Testing if container is able to run from image */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
    stage('Deploy image') {
        /* Deploys passed builds to Kubernetes without disrupting service */

        sshagent(['my-ssh-key']) {
            sh 'ssh ubuntu@34.227.11.210 kubectl set image deployments/cw2 cw2=zumar201/cw2:$BUILD_NUMBER'
        }
    }  
}
