pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "sohanmer/train-schedule-autodeploy"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                //archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps { 
		            sh '/usr/local/bin/docker build -t sohanmer/train-schedule .'
                }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                // script {
                //     docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                //         app.push("${env.BUILD_NUMBER}")
                //         app.push("latest")
                //     }
                // }
                sh '/usr/local/bin/docker push sohanmer/train-schedule-autodeploy'
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                // kubernetesDeploy(
                //     kubeconfigId: 'k8sconfig',
                //     configs: 'train-schedule-kube-canary.yml',
                //     kubeConfig: [path: '/Users/sohanmer/.jenkins/workspace/.kube/config'],
                //     enableConfigSubstitution: true
                // )
                withEnv(["KUBECONFIG=/Users/sohanmer/.jenkins/workspace/.kube/config"]) {
                    sh "/opt/homebrew/bin/kubectl apply -f train-schedule-kube-canary.yml"
                }

            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'k8sconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    kubeConfig: [path: '/Users/sohanmer/.jenkins/workspace/.kube/config'],
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'k8sconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
