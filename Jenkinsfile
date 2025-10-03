pipeline {
    agent any
    environment {
        PROJECT_DIR        = 'src/FullstackApp'
        PROJECT            = 'FullstackApp.csproj'
        CONFIGURATION      = 'Release'
        PUBLISH_DIR        = 'publish'
        ARTIFACT           = "FullstackApp-${BUILD_NUMBER}.zip"
        SONAR_PROJECT_KEY  = 'devops-project'
        SONAR_PROJECT_NAME = 'DevOps FullstackApp'
        AZURE_RG           = credentials('azure-reg-creds')
        NEXUS_DNS          = credentials('nexus-dns-creds')
        SONAR_HOST_URL     = credentials('sonar-host-creds')
    }


    stages {
        stage('Fetch Code') {
            steps{
                git branch: 'Michael-stg', url: 'https://github.com/darisjerez/devops-class-dontnet-fullstack.git' 
            }
        }
        
        stage('Restore') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh "dotnet restore ${PROJECT}"
                }
            }
        }

        stage('Static Analysis / lint') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh "dotnet format --verify-no-changes"
                }
            }
        }

        stage('Test and Code Coverage') {
            steps {
                dir("tests/FullstackApp.Tests") {
                    sh "dotnet test --collect:'XPlat Code Coverage' --verbosity normal"
                }
            } 
            post{
                always {
                    archiveArtifacts artifacts: "tests/FullstackApp.Tests/TestResults/**/*.cobertura.xml", allowEmptyArchive: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    withSonarQubeEnv('sonarserver') {
                        dir("${PROJECT_DIR}") {
                            sh """
                            export PATH="\$HOME/.dotnet/tools:\$PATH"

                            dotnet sonarscanner begin \\
                              /k:"${SONAR_PROJECT_KEY}" \\
                              /n:"${SONAR_PROJECT_NAME}" \\
                              /v:"1.0" \\
                              /d:sonar.host.url=${SONAR_HOST_URL} \\
                              /d:sonar.login=\${SONAR_AUTH_TOKEN} \\
                              /d:sonar.cs.opencover.reportsPaths="${WORKSPACE}/tests/FullstackApp.Tests/TestResults/coverage/coverage.opencover.xml"
                              
                            dotnet build ${PROJECT} -c Release

                            dotnet sonarscanner end \\
                              /d:sonar.login=\${SONAR_AUTH_TOKEN}
                            """
                        }
                    }
                }
            }
        }


        stage('Publish') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh """
                    dotnet publish ${PROJECT} -c ${CONFIGURATION} -o ${PUBLISH_DIR} --no-build
                    cd ${PUBLISH_DIR}
                    zip -r ../${ARTIFACT} .
                    """
                }
            }
        }

        stage('Upload Nexus Artifact') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_DNS}",
                        groupId: 'com.babel.dotnet',
                        version: "${BUILD_NUMBER}",
                        repository: 'dotnet-releases',
                        credentialsId: 'nexuslogin',
                        artifacts: [[
                            artifactId: 'FullstackApp',
                            classifier: '',
                            file: "${PROJECT_DIR}/${ARTIFACT}",
                            type: 'zip'
                        ]]
                    )
                }
            }
        }
    }
}