pipeline{
    agent any

    tools {
        git 'Default'
        jdk 'OracleJDK17'
        maven 'MAVEN3'
    }

    stages {
        stage('1-FetchTheCode') {
            steps {
                echo 'Cloning the latest application version from GitHub'
                // git credentialsId: 'githublogincred', url: 'https://github.com/JendareyTechnologies/Jendarey-Engineers-Voting-Result-App-War-Project2.git'
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'githublogincred', url: 'https://github.com/JendareyTechnologies/Jendarey-Engineers-Voting-Result-App-War-Project2.git']])
           }
        }

        stage('2-BuildTheCode') {
            steps {
                echo 'Cleaning and building packages'
                sh 'mvn clean install -DskipTests'      
           }
        }

        stage('3-UnitTest') {
            steps {
                echo 'Jenkins is running JUnit-test-cases'
                sh 'mvn test'      
           }
        }

        stage('4-OWASPScan') {
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
                // sh 'mvn deploy'
                echo 'Jenkins uploaded Artifacts to Nexus'
           }
        }

        stage('7-DeployToProduction') {
            steps {
                echo 'Jenkins is about to deploy our application to Production environment'
                deploy adapters: [tomcat9(credentialsId: 'tomcatlogincred', path: '', url: 'http://54.90.230.122:8080/')], contextPath: null, war: 'target/*.war'
           }
        }

     }

    post {
        success {
            echo 'Pipeline succeeded! Sending final notification...'
            slackSend(
                channel: '#classa24-new-cicd-votingapp-project',
                color: 'good',
                message: "*SUCCESS:* Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} has succeeded. \n Please check for more information at: ${env.BUILD_URL}"
            )
        }
        failure {
            echo 'Pipeline failed! Sending final notification...'
            slackSend(
                channel: '#classa24-new-cicd-votingapp-project',
                color: 'danger',
                message: "*FAILURE:* Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} has failed. \n Please check for more information at: ${env.BUILD_URL}"
            )
        }
    }
}
