def COLOR_MAP = [
    SUCCESS: 'good' ,
    FAILURE: 'danger'
]

pipeline {
    agent any

    tools {
        git 'Default'
        jdk 'OracleJDK17'
        maven 'MAVEN3'
    }

    stages {
        stage('1-Fetch the Code') {
            steps {
                echo 'Cloning the latest application version from GitHub'
                // git credentialsId: 'gitlogincred', url: 'https://github.com/JendareyTechnologies/Jendarey-Engineers-Voting-Result-App-War-Project2.git'
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitlogincred', url: 'https://github.com/JendareyTechnologies/Jendarey-Engineers-Voting-Result-App-War-Project2.git']])

            }
        }

        stage('2-Build the Code') {
            steps {
                echo 'Cleaning and building packages'
                sh 'mvn clean install -DskipTests'
            }
            post {
                always {
                    echo 'Archiving artifacts now'
                    archiveArtifacts allowEmptyArchive: true, artifacts: '**/*.war', followSymlinks: false, onlyIfSuccessful: true
                 }
            }

        }

        stage('3-Unit Test') {
            steps {
                echo 'Cleaning and building packages'
                sh 'mvn test'
            } 
        }

        stage('4-OWASP Scan') {
            steps {
                echo 'Jenkins is setting up Owasp-Scan'
                echo 'Jenkins is performing Owasp-Scan Analysis'
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DependencyCheck'
            } 
        } 

        stage('5-CodeQualityAnalysis') {
            steps {
                echo 'Jenkins is setting up SonarQube authentication'
                echo 'Jenkins is performing Code Quality Analysis'
                sh 'mvn sonar:sonar'

            } 
        }

        stage('6-UploadArtifacts') {
            steps {
                echo 'Jenkins is configuring Nexus authentication'
                sh 'mvn deploy'
                echo 'Jenkins uploaded Artifacts to Nexus'

            } 
        }

        stage('7-Deploy to UAT') {
            steps {
                echo 'Jenkins is about to deploy our application to User Acceptance Testing environment'
                deploy adapters: [tomcat9(credentialsId: 'newtomcatlogincred', path: '', url: 'http://52.90.115.49:8080/')], contextPath: null, war: 'target/*.war'

            } 
        }

        stage('8-Approval') {
            steps {
                echo 'votingapp application is ready for review'
                timeout(time: 1, unit: 'HOURS') {
                    input 'voting application is ready for review and approval'
                }
            } 
        }

        stage('9-Deploy to Production') {
            steps {
                echo 'Jenkins is about to deploy our application to User Acceptance Testing environment'
                deploy adapters: [tomcat9(credentialsId: 'newtomcatlogincred', path: '', url: 'http://52.90.115.49:8080/')], contextPath: null, war: 'target/*.war'

            } 
        }

        stage('10-Notification-Slack') {
            steps {
                echo 'Jenkins is about to send Slack Notification'
                slackSend(
                    channel: '#classa24-cicd-votingapp-project',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} \n Please check for more information at: ${env.BUILD_URL}" 
                )

            } 
        }

    }

}
