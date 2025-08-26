pipeline {
  agent any
  tools { nodejs "Node20" }

  parameters {
    string(name: 'AZ_RG',  defaultValue: 'rg-node-mongo-canadacentral', description: 'Azure resource group')
    string(name: 'AZ_APP', defaultValue: 'lucky148715', description: 'Azure Web App name')
    booleanParam(name: 'SET_APPSETTINGS_ONCE', defaultValue: true, description: 'Set SCM_DO_BUILD_DURING_DEPLOYMENT the first time')
  }

  triggers { githubPush() }

  options { ansiColor('xterm'); timestamps(); durabilityHint('PERFORMANCE_OPTIMIZED') }

  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Install dependencies') { steps { sh 'npm ci' } }
    stage('Package app (zip)') {
      steps {
        sh '''
          rm -f app.zip || true
          zip -r app.zip . -x ".git/*" ".github/*" "node_modules/*" "tests/*"
          ls -lh app.zip
        '''
        archiveArtifacts artifacts: 'app.zip', fingerprint: true
      }
    }
    stage('Azure Login') {
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZ_APP_ID', passwordVariable: 'AZ_PASSWORD'),
          string(credentialsId: 'azure-tenant', variable: 'AZ_TENANT')
        ]) {
          sh '''
            az logout || true
            az login --service-principal -u "$AZ_APP_ID" -p "$AZ_PASSWORD" --tenant "$AZ_TENANT"
            az account show
          '''
        }
      }
    }
    stage('App settings (first run)') {
      when { expression { return params.SET_APPSETTINGS_ONCE } }
      steps {
        sh '''
          az webapp config appsettings set \
            --resource-group "$AZ_RG" --name "$AZ_APP" \
            --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true
        '''
      }
    }
    stage('Deploy to Azure App Service') {
      steps {
        sh 'az webapp deploy --resource-group "$AZ_RG" --name "$AZ_APP" --src-path app.zip'
      }
    }
  }
  post { always { cleanWs() } }
}
