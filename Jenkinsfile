pipeline {
    agent { 
        node { 
        label 'roboshop' 
        } 
    }
    environment { 
              appVersion   = "" 
              acc_id = "971126626665"
              region = "us-east-1"
    }
    options { 
        //disableConcurrentBuilds() 
        timeout(time: 5, unit: 'MINUTES')
    } 
    /* parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'DEPLOY', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    } */
    stages {
        stage('Read Version') {
            steps {
                script {
                    // Load and parse the JSON file
                    def packageJson = readJSON file: 'package.json'
                    
                    // Access fields directly
                    appVersion = packageJson.version
                    echo "Building version ${appVersion}"
                }
            }
        }
        stage('Installing Dependencies') {
            steps {
                script{
                    sh """
                        npm install
                    """
                }
            }
        }

        stage('unit tests') {
            steps {
                script{
                    sh """
                        npm test
                    """
                }
            }
        }

        stage('sonarqube analysis') {
            steps {
                script{
                    def scannerHome = tool name: 'sonar-8' // agent configuration
                    withSonarQubeEnv('sonar-server') { // analysing and uploading to server
                            sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }    
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Building Image') {
            steps {
                script{
                    withAWS(credentials: 'aws-creds', region: "${region}") {
                        // Commands here have AWS authentication
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.${region}.amazonaws.com

                            docker build -t ${acc_id}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} .

                            docker push ${acc_id}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}
                        """
                    }
                }
            }
        }
    }    
    // post-build
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
        // here we can give mail id & slack to notify when pipeline is failed or success
        success {
            echo 'pipeline success'
        }
        failure {
            echo 'pipeline failure'
        }
    }
}  
