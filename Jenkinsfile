pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage ('Initial') {
            steps {
              echo '========================================='
              echo '                INITIAL '
              echo '========================================='
              sh '''
                   echo "PATH = ${PATH}"
                   echo "M2_HOME = ${M2_HOME}"
               '''
            }
        }
		stage ('CheckOut'){
            steps {
				echo '========================================='
                echo '                CHECK OUT GIT '
                echo '========================================='
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/pop3SG/spring-boot-kubernetes']]])
            }
        }
        stage ('Compile') {
            steps {
                echo '========================================='
                echo '                COMPILE '
                echo '========================================='
                 sh 'mvn clean compile -e'
            }
        }
        stage('SonarQube analysis') {
           steps{
                echo '========================================='
                echo '                SONARQUBE '
                echo '========================================='
                script {
                    def scannerHome = tool 'SonarQube Scanner';
                    withSonarQubeEnv('Sonar Server') {
                      sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=jenkins-pipeline -Dsonar.sources=target/ -Dsonar.host.url=http://172.18.0.3:9000 -Dsonar.login=34e68840680ac8bcf783630fdd32dd251aafedb9"
                    }
                }
           }
        }
	 stage ('SCA') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check'
                dependencyCheckPublisher failedNewCritical: 5, failedTotalCritical: 10, pattern: 'terget/dad.xml', unstableNewCritical: 3, unstableTotalCritical: 5
            }
        }    
    }
}
