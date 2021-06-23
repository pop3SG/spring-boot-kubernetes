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
                      sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Tarea4 -Dsonar.host.url=http://172.18.0.3:9000 -Dsonar.login=288fdc72da55397274d2cdf0241890a60715295e"
                    }
                }
           }
        }
    }
}
