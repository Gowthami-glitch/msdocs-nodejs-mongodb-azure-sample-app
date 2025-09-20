pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS 20', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
        AZURE_SP = credentials('azure-sp')
        AZURE_TENANT = credentials('azure-tenant')
        RESOURCE_GROUP = 'Project11RG'
        WEB_APP_NAME = 'lucky148715'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/Gowthami-glitch/msdocs-nodejs-mongodb-azure-sample-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                withEnv(["PATH+NODE=${NODEJS_HOME}/bin"]) {
                    sh 'npm install'
                }
            }
        }

        stage('Package App') {
            steps {
                sh 'zip -r app.zip *'
            }
        }

        stage('Publish Artifact') {
            steps {
                archiveArtifacts artifacts: 'app.zip', fingerprint: true
            }
        }

        stage('Azure Login') {
            steps {
                sh '''
                az logout || true
                az login --service-principal -u "$AZURE_SP_USR" -p "$AZURE_SP_PSW" --tenant "$AZURE_TENANT" --output none
                '''
            }
        }

        stage('Deploy to Azure Web App') {
            steps {
                sh '''
                az webapp deploy \
                  --resource-group $RESOURCE_GROUP \
                  --name $WEB_APP_NAME \
                  --src-path app.zip \
                  --type zip
                '''
            }
        }
    }

    post {
    success {
        echo "✅ Deployment successful!"
        echo "🌐 Visit your app at: http://lucky148715-fgbxfxehg7frcuhp.canadacentral-01.azurewebsites.net"
    }
    failure {
        echo "❌ Deployment failed. Please check logs."
    }
}

}
