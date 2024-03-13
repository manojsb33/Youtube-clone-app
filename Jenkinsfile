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
        SCANNER_HOME=tool 'sonar-scaner'
    }
    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'manoj3366', description: 'Docker Hub Username')
        string(name: 'IMAGE_NAME', defaultValue: 'youtube', description: 'Docker Image Name')
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
        stage('OWASP FS SCAN') {
        when { expression { params.action == 'create'}}
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy file scan'){
        when { expression { params.action == 'create'}}    
            steps{
                TrivyFileScan()
            }
        }
        stage('Docker Build'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                   def dockerHubUsername = params.DOCKER_HUB_USERNAME
                   def imageName = params.IMAGE_NAME

                   DockerBuild(dockerHubUsername, imageName)
                }
            }
        }
        stage('Trivy iamge'){
        when { expression { params.action == 'create'}}    
            steps{
                TrivyImage()
            }
        }
        stage('Run container'){
        when { expression { params.action == 'create'}}    
            steps{
                RunContainer()
            }
        }
        stage('Remove container'){
        when { expression { params.action == 'delete'}}    
            steps{
                RemoveContainer()
            }
        }
        stage('Kube deploy') {
        when { expression { params.action == 'create'}}
            steps{
                KubeDeploy()
            }    
        }
        stage('delete deploy') {
        when { expression { params.action == 'delete'}}
            steps{
                KubeDelete()
            }    
        }
    }
    post {
        always {
            echo 'Slack Notification'
            slackSend (
                channel: '#jenkins', 
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            )
        }

    }
}























    
    
    
    
    
    
    
    
    
    
    
    
    
