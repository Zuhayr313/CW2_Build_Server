node {
    def app

    stage('Clone repository') {
        /* Cloning the repository to the workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("zumar201/cw2")
    }

    stage('Test image') {
        /* Testing if container is able to run from image */

        sh 'echo "Running Container Using cw2 Image"'
        sh 'docker container run --detach --publish 80:80 --name cw2 zumar201/cw2:1.0'
        
        sh 'echo "Ensuring Container Launched Successfully"'
        sh 'docker container ls'
        
        sh 'echo "Stopping and Removing cw2 Container"'
        sh 'docker container stop cw2'
        sh 'dokcer container rm cw2'
        
        sh 'echo "Tests passed"'
        
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
            sh 'ssh ubuntu@3.93.163.156 kubectl set image deployments/cw2 cw2=zumar201/cw2:$BUILD_NUMBER'
        }
    }  
}
