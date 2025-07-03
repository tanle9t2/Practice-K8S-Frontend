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
        RUN_PIPELINE = "false"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: env.BRANCH, url: env.REPO
            }
        }

        stage("Sonarqube Check") {
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

        stage('Detect Changes in src/') {
            steps {
                script {
                    def changedFiles = sh(
                            script: "git diff --name-only HEAD~1 HEAD",
                            returnStdout: true
                    ).trim()
                    if (changedFiles.readLines().any { it.startsWith('src/') }) {
                        env.RUN_PIPELINE = "true"
                    } else {
                        env.RUN_PIPELINE = "false"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            when {
                expression { env.RUN_PIPELINE == "true" }
            }
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression { env.RUN_PIPELINE == "true" }
            }
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage("Update Manifest") {
            when {
                expression { env.RUN_PIPELINE == "true" }
            }
            steps {
                echo '🔄 Updating K8s Manifest (via SSH)'
                sshagent(credentials: ['github-ssh-key']) {
                    sh '''
                        rm -rf Practice-K8S-Frontend-Config
                        git config --global user.name "tanle9t2"
                        git config --global user.email "fcletan12@gmail.com"

                        mkdir -p ~/.ssh
                        ssh-keyscan github.com >> ~/.ssh/known_hosts

                        git clone git@github.com:tanle9t2/Practice-K8S-Frontend-Config.git

                        cd Practice-K8S-Frontend-Config
                        sed -i "s|tag: .*|tag: ${IMAGE_TAG}|" helm/values.yaml
                        git add helm/values.yaml
                        git commit -m "Update image tag to ${IMAGE_TAG}" || true
                        git push origin main
                    '''
                }
            }
        }
    }
}
