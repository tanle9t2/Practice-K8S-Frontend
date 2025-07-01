pipeline {
    agent any

    environment {
        CI = true
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "tanle92/react-app"
        BRANCH = "main"
        REPO = 'https://github.com/tanle9t2/Practice-K8S-Frontend.git'
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: env.BRANCH, url: env.REPO
            }
        }

        stage("OWASP Scan") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonarqube Check"){
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Frontend-React \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Frontend-React
                '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} react-frontend/"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }


            }
        }
    }
}
