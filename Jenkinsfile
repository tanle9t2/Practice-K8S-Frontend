pipeline {
    agent any
    tools {
        nodejs 'Node17'
    }
    environment {
        CI = true
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "tanle92/react-app"
        BRANCH = "main"
        REPO = 'https://github.com/tanle9t2/Practice-K8S-Frontend.git'
        REPO_CONFIG = "https://github.com/tanle9t2/Practice-K8S-Frontend-Config.git"
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
        stage('Set Commit-Based Tag') {
            steps {
                script {
                    def commit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "main-${commit}"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
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
        stage("Update Manifest") {
            steps {
                echo 'Updating Manifest'

                withCredentials([
                        string(credentialsId: 'GIT_USERNAME', variable: 'GIT_USERNAME'),
                        string(credentialsId: 'GIT_EMAIL', variable: 'GIT_EMAIL')
                ]) {
                    sh '''
                git config user.name "$GIT_USERNAME"
                git config user.email "$GIT_EMAIL"

                git clone ${REPO_CONFIG}
                cd Practice-K8S-Frontend-Config/helm
                sed -i "s|tag: .*|tag: ${IMAGE_TAG}|" values.yaml
                git add values.yaml
                git commit -m "Update image tag to ${IMAGE_TAG}"
                git push origin main
            '''
                }
            }
        }

    }
}
