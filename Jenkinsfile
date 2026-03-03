pipeline {
    agent any
environment {
        DOCKER_IMAGE = "spring-k8s-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_CONFIG_CREDENTIALS = "kubeconfig"  // Add your kubeconfig in Jenkins credentials
    }
    stages {
	stage('Git Checkout') {
            steps {
                  git branch: 'main', url: 'https://github.com/Laouini-islem/mon-devops.git'
            }
        }
		
	stage('Maven Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
		stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t $DOCKER_USER/$DOCKER_IMAGE:$IMAGE_TAG .
                        docker tag $DOCKER_USER/$DOCKER_IMAGE:$IMAGE_TAG $DOCKER_USER/$DOCKER_IMAGE:latest
                        docker push $DOCKER_USER/$DOCKER_IMAGE:$IMAGE_TAG
                        docker push $DOCKER_USER/$DOCKER_IMAGE:latest
                    '''
                }
            }
        }
    
    stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
                echo "Using kubeconfig at $KUBECONFIG"
                kubectl get nodes
                kubectl apply -f k8s/deployment.yaml
            '''
        }
    }
}

}
		    
}
