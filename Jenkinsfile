pipeline {
    agent any
    

    parameters {
        string(
            name: 'FRONTEND_DOCKER_TAG',
            defaultValue: '',
            description: 'Docker image tag for frontend'
        )

        string(
            name: 'BACKEND_DOCKER_TAG',
            defaultValue: '',
            description: 'Docker image tag for backend'
        )
    }

    environment {
        PROJECT_NAME = 'wanderlust'
        SONAR_PROJECT_KEY = 'wanderlust-mykey_wanderlust'
    }

    stages {

        stage("Validate Parameters") {
            steps {
                script {
                    if (!params.FRONTEND_DOCKER_TAG?.trim() ||
                        !params.BACKEND_DOCKER_TAG?.trim()) {

                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage("Git Code Checkout") {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'git-token',
                        url: 'https://github.com/KevinJCloud/Wanderlust_3TIER.git'
                    ]]
                )
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan ./',
                    odcInstallation: 'owasp'
                )

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }

        stage("SonarCloud Analysis") {
            steps {
               script {
    def scannerHome = tool 'sonarqube-scanner'

    withSonarQubeEnv("sonar-qube") {
        sh """
            ${scannerHome}/bin/sonar-scanner \
            -Dsonar.projectName=wanderlust \
            -Dsonar.projectKey=wanderlust-mykey_wanderlust \
            -Dsonar.organization=wanderlust-mykey \
            -X
        """
    }
}
            }
        }

       

        stage("Exporting Environment Variables") {
            parallel {

                stage("Backend Env Setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatebackendnew.sh"
                        }
                    }
                }

                stage("Frontend Env Setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatefrontendnew.sh"
                        }
                    }
                }
            }
        }

        stage("Docker: Build Images") {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker login',
                            usernameVariable: 'Dockerhub_user',
                            passwordVariable: 'Dockerhub_pass'
                        )
                    ]) {

                        dir('backend') {
                            sh """
                                docker build \
                                -t ${Dockerhub_user}/${PROJECT_NAME}:${params.BACKEND_DOCKER_TAG} .
                            """

                            sh """
                                trivy image \
                                ${Dockerhub_user}/${PROJECT_NAME}:${params.BACKEND_DOCKER_TAG}
                            """
                        }

                        dir('frontend') {
                            sh """
                                docker build \
                                -t ${Dockerhub_user}/${PROJECT_NAME}:${params.FRONTEND_DOCKER_TAG} .
                            """

                            sh """
                                trivy image \
                                ${Dockerhub_user}/${PROJECT_NAME}:${params.FRONTEND_DOCKER_TAG}
                            """
                        }
                    }
                }
            }
        }

        stage("Docker Push") {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker login',
                            usernameVariable: 'Dockerhub_user',
                            passwordVariable: 'Dockerhub_pass'
                        )
                    ]) {

                        sh '''
                            echo "$Dockerhub_pass" | docker login \
                            -u "$Dockerhub_user" \
                            --password-stdin
                        '''

                        sh """
                            docker push ${Dockerhub_user}/${PROJECT_NAME}:${params.FRONTEND_DOCKER_TAG}
                        """

                        sh """
                            docker push ${Dockerhub_user}/${PROJECT_NAME}:${params.BACKEND_DOCKER_TAG}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts(
                artifacts: '*.xml',
                followSymlinks: false
            )

            build job: "CD",
                parameters: [
                    string(
                        name: 'FRONTEND_DOCKER_TAG',
                        value: "${params.FRONTEND_DOCKER_TAG}"
                    ),
                    string(
                        name: 'BACKEND_DOCKER_TAG',
                        value: "${params.BACKEND_DOCKER_TAG}"
                    )
                ]
        }
    }
}
