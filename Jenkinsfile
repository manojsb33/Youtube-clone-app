@Library('Jenkins-shared-library') _
def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]


pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWorkspace()
            }
        }
        stage('checkout from Git'){
            steps{
                GitCheckout('https://github.com/manojsb33/Youtube-clone-app.git', 'main')
            }
        }
     }
     post {
         always {
             echo 'Slack Notifications'
             slackSend (
                 channel: '#jenkins', 
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
               )
           }
        stage('sonarqube Analysis'){
        when { expression { params.action == 'create'}}    
            steps{
                SonarqubeAnalysis()
            }
        }
        stage('sonarqube QualitGate'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                    def credentialsId = 'Sonar-token'
                    QualityGate(credentialsId)
                }
            }
        }
        stage('Npm'){
        when { expression { params.action == 'create'}}    
            steps{
                NpmInstall()
            }
        }
    }
}
