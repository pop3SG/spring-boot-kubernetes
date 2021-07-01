pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage ('Initial') {
         	steps {
              		figlet ' INICIO '
              		sh '''
                   	echo "PATH = ${PATH}"
                   	echo "M2_HOME = ${M2_HOME}"
               		'''
            }
        }
	stage ('CheckOut'){
            	steps {
			figlet ' CHECK OUT GIT '
                	checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/pop3SG/spring-boot-kubernetes']]])
            	}
        }
        stage ('Compile') {
        	steps {
                	figlet 'Compilar Aplicacion'
                 sh 'mvn clean compile -e'
            }
        }
        stage('SonarQube analysis') {
           	steps{
           		figlet 'Scan SonarQube'
                	script {
                		def scannerHome = tool 'SonarQube Scanner';
	                	withSonarQubeEnv('Sonar Server') {
        	        		//sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=tarea4 -Dsonar.sources=target/ -Dsonar.host.url=http://172.18.0.3:9000 -Dsonar.login=f74fc670543b5f3d217066fcdd8340ec592be0cd"
                 			sh '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=tarea4 -Dsonar.sources=target/ -Dsonar.host.url=http://172.18.0.4:9000 -Dsonar.login=6cf1646888a00d9010075217b6df707e0dd9a459'
                    }
                }
           }
        }
	stage ('SCA') {
        	steps {
		figlet 'SCA'
                	sh 'mvn org.owasp:dependency-check-maven:check'
                	dependencyCheckPublisher failedNewCritical: 5, failedTotalCritical: 10, pattern: 'dependency-check-report.xml', unstableNewCritical: 3, unstableTotalCritical: 5
            }
        } 
	stage('Scan Docker') {
		steps {
			figlet 'Scan docker'
			script {
				//Variables Docker	
				env.DOCKER = tool 'Docker';
				env.DOCKER_EXEC = "${DOCKER}/bin/docker"
				//muestra variable Docker_exec 	
				echo "${DOCKER_EXEC}"
				// descargar dcker trivy
				sh '${DOCKER_EXEC} pull aquasec/trivy:0.18.3'
				//scaner docker git de la tarea 7
				sh '${DOCKER_EXEC} run --rm -v /var/run/docker.sock:/var/run/docker.sock -v /home/arqcloud/owasp-zap:/root/.cache/ aquasec/trivy:0.18.3 repo https://github.com/PheaSoy/spring-boot-kubernetes'
				//scaner docker git de ejemplo https://github.com/knqyf263/trivy-ci-test 
				//sh '${DOCKER_EXEC} run --rm -v /var/run/docker.sock:/var/run/docker.sock -v /home/arqcloud/owasp-zap:/root/.cache/  aquasec/trivy:0.18.3 https://github.com/knqyf263/trivy-ci-test'
				//scaner docker anaizando docker vulnerables/web-dvwa 
				//sh '${DOCKER_EXEC} run --rm -v /var/run/docker.sock:/var/run/docker.sock -v /home/arqcloud/owasp-zap:/root/.cache/  aquasec/trivy:0.18.3 vulnerables/web-dvwa'
			}
		}
	} 
	stage('ZAP') {
        	steps {
        	    figlet 'Owasp Zap DAST'
        		script {
			    	//Variables Docker	
        		    	env.DOCKER = tool 'Docker';
		            	env.DOCKER_EXEC = "${DOCKER}/bin/docker"
			    	//muestra variable Docker_exec 	
			    	echo "${DOCKER_EXEC}"
				//elimina imagen docker zap2
        		    	sh '${DOCKER_EXEC} rm -f zap2'
				// descraga la version estable de zap
        		    	sh '${DOCKER_EXEC} pull owasp/zap2docker-stable'
                            	//Levantar el zap en modo escucha en el puerto 8090
				sh '${DOCKER_EXEC} run --add-host="localhost:192.168.23.134" --rm -e LC_ALL=C.UTF-8 -e LANG=C.UTF-8 --name zap2 -u zap -p 8090:8080 -d owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true'
                            	 // ahora ejecutamos el full-scan a la pagina http://zero.webappsecurity.com y el reporte se guarda en /home/arqcloud/owasp-zap
				sh '${DOCKER_EXEC} run --add-host="localhost:192.168.23.134" -v /home/arqcloud/owasp-zap:/zap/wrk/:rw --rm -i owasp/zap2docker-stable zap-full-scan.py -t "http://zero.webappsecurity.com" -I -r zap_full_scan_report2.html -l PASS'
        		}
        	}
        }    
    }
}
