pipeline{
    agent any
    environment{
        PROJECT_DIR        = 'src/FullstackApp'
        PROJECT            = 'FullstackApp.csproj'
        CONFIGURATION      = 'Release'
        PUBLISH_DIR        = 'publish'
        ARTIFACT           = "FullstackApp-${BUILD_NUMBER}.zip"
        SONAR_PROJECT_KEY  = 'devops-project'
        SONAR_PROJECT_NAME = 'DevOps FullstackApp'
        AZURE_APP_NAME_QA = 'dotnet-app-24242342'
        TEMPLATE_FILE = './ARM/webapp-template.json'
        QA_PARAMETERS_FILE = './ARM/qa-parameters.json'
        NEXUS_DNS          = credentials('nexus-dns-creds')
        AZURE_RG           = credentials('azure-reg-creds')

    }
    stages{
        stage('Download Artifact') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'nexuslogin', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    curl -u $USER:$PASS -O \
                    http://${NEXUS_DNS}/repository/dotnet-releases/com/babel/dotnet/FullstackApp/${BUILD_NUMBER}/FullstackApp-${BUILD_NUMBER}.zip

                    """
                }
            }
        }

        stage('Login to Azure'){
            steps{
                withCredentials([azureServicePrincipal('az-creds')]){
                    sh '''
                       az login --service-principal \
                         -u $AZURE_CLIENT_ID \
                         -p $AZURE_CLIENT_SECRET \
                         --tenant $AZURE_TENANT_ID
                       az account set --subscription $AZURE_SUBSCRIPTION_ID
                    '''
                }
            }
        }

        stage ('Deploying ARM') {
            steps{
                sh """
                echo "Verificando si existen los archivos ARM..."
                ls -la ./ARM

                az deployment group create \
                  --resource-group ${AZURE_RG} \
                  --template-file ${TEMPLATE_FILE} \
                  --parameters ${QA_PARAMETERS_FILE} \
                  --debug
                """
            }
        }
        stage('Deploy to QA'){
            steps{
                sh """
                az webapp deployment source config-zip \
                  --resource-group ${AZURE_RG} \
                  --name ${AZURE_APP_NAME_QA} \
                  --src ${ARTIFACT}
                """
            }
        }
    }
}