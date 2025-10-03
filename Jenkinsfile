pipeline {
    agent any
    parameters {
        choice(name: 'ENV_TO_DEPLOY', choices: ['ci-pipeline', 'release-QA', 'release-PROD'], description: 'Branch del pipeline hijo a ejecutar')
    }
    
    stages {

        stage('Trigger CI pipeline') {
            when {expression {params.ENV_TO_DEPLOY == 'ci-pipeline'}}
            steps{
                script{
                    def qaPipeline = 'donet-project-ci'
                    echo "Disparando CI pipeline: ${qaPipeline}"
                    try{
                        build job: qaPipeline,
                              parameters: [
                                string(name: 'BRANCH', value: 'release-QA'),
                                string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                              ],
                              wait: true
                    } catch (err) {
                        echo "Failed: ${err}"
                        error "Deteniendo pipeline padre por fallo en QA"
                    }
                }
            }
        }

        stage('Deploy to QA') {
            when {expression {params.ENV_TO_DEPLOY == 'release-QA'}}
            steps{
                script{
                    def qaPipeline = 'Dotnet-project-QA'
                    echo "Disparando pipeline QA: ${qaPipeline}"
                    try{
                        build job: qaPipeline,
                              parameters: [
                                string(name: 'BRANCH', value: 'release-QA'),
                                string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                              ],
                              wait: true
                    } catch (err) {
                        echo "Failed: ${err}"
                        error "Deteniendo pipeline padre por fallo en QA"
                    }
                }
            }
        }

        stage('Deploy to PROD') {
            when {expression {params.ENV_TO_DEPLOY == 'release-PROD' } }
            steps {
                script {
                    def prodPipeline = 'Dotnet-project-PROD'
                    echo "Disparando pipeline PROD: ${prodPipeline}"
                    try{
                        build job: prodPipeline,
                              parameters: [
                                string(name: 'BRANCH', value: 'release-PROD'),
                                string(name: 'BUILD_NUMBER_PARENT', value: "${BUILD_NUMBER}")
                              ],
                              wait: true
                    } catch (err) {
                        echo "Failed: ${err}"
                        error "Deteniendo pipeline padre"
                    }
                }
            }
        }
    }
}