pipeline {
    agent any
    
    stages {
        stage('Trigger CI pipeline') {
            steps {
                script {
                    def ciPipeline = 'Dotnet-project-CI'
                    echo "Disparando CI pipeline: ${ciPipeline}"
                    build job: ciPipeline,
                          parameters: [
                            string(name: 'BRANCH', value: 'ci-pipeline'),
                            string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                          ],
                          wait: true
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    def qaPipeline = 'Dotnet-project-QA'
                    echo "Disparando pipeline QA: ${qaPipeline}"
                    build job: qaPipeline,
                          parameters: [
                            string(name: 'BRANCH', value: 'release-QA'),
                            string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                          ],
                          wait: true
                }
            }
        }

        stage('Deploy to PROD') {
            steps {
                script {
                    def prodPipeline = 'Dotnet-project-PROD'
                    echo "Disparando pipeline PROD: ${prodPipeline}"
                    build job: prodPipeline,
                          parameters: [
                            string(name: 'BRANCH', value: 'release-PROD'),
                            string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                          ],
                          wait: true
                }
            }
        }
    }
}
