pipeline {
    agent any
    environment { 
        registry = "bhatanimesh02/jenkins-ci" 
        registryCredential = 'dockerhub' 
        dockerImage = '' 
        tag = sh(returnStdout: true, script: "git rev-parse --short=4 HEAD").trim()
        PROJECT_ID = 'concrete-zephyr-343510'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'kubernetes'
        
    }
    tools{
        maven "MAVEN"
    }

    stages {
        stage('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/animesh-workspace/jenkins-gke-CI-CD.git', credentialsId: '67a57de4-27ae-4ab2-b0f8-38f84a81bba6']]])
            }
        }
        stage('maven build') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }
        stage('Building our image') { 
            steps { 
                script { 
                    dockerImage = docker.build registry + ":$tag" 
                }
            } 
        }
        stage('Deploy our image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                } 
            }
        } 
        stage('Deploy to K8s') {
		    steps{
			    echo "Starting to deploy..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${tag}/g' deployment.yaml"
			    echo "Start deployment of deployment.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deploy.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
        }
    }
}
