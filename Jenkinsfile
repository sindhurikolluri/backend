pipeline {
    agent { label 'AGENT-1' }
    environment { 
        PROJECT = 'expense'
        COMPONENT = 'backend'
        appVersion = ''
         ACC_ID = '717279725557'
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage('Read Version') {
            steps {
               script{
                 def packageJson = readJSON file: 'package.json'
                 appVersion = packageJson.version
                 echo "Version is: $appVersion"
               }
            }
        }
        stage('Install Dependencies') {
            steps {
               script{ 
                 sh """
                    npm install
                 """
               }
            }
        }
         stage('Docker Build') {
            steps {
               script{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                    docker build -t ${PROJECT}/${COMPONENT} .
                    docker tag expense/backend:latest ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                    docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                    """
                }
                 
               }
            }
    }
     stage('Trigger Deploy'){
            when { 
                expression { params.deploy }
            }
            steps{
                build job: 'backend-cd', parameters: [string(name: 'version', value: "${appVersion}")], wait: true
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'I will run when pipeline is failed'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
    }
}