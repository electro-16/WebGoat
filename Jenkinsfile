pipeline {
  agent any 
  tools {
    maven 'Maven'
  }  
  stages {
   stage ('Initialize') {
      steps {
        sh '''
                  echo "PATH = ${PATH}"
                  echo "M2_HOME = ${M2_HOME}"
           '''
      }
    }
    stage ('Check-Git-Secrets') {
      steps {
        sh ' trufflehog3 -f json https://github.com/electro-16/webapp.git -o trufflehog_output.json || true '
      }
    }
    // stage ('Source Composition Analysis') {
    //   steps {
    //      sh 'rm owasp* || true '
    //      sh 'wget "https://raw.githubusercontent.com/electro-16/webapp/master/owasp-dependency-check.sh" '
    //      sh 'chmod +x owasp-dependency-check.sh'
    //      sh 'bash owasp-dependency-check.sh'
    //      sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml' 
    //   }
    // }

   
    stage('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
    
  
  stage ('Static analysis') {
    steps { 
      withSonarQubeEnv('sonar') {
        sh 'mvn sonar:sonar'
	//sh './sonarqube_report.sh'
        }
      }
    }
	  
  stage ('Generate build') {
      steps {
        sh 'mvn clean install -DskipTests'
      }
    }  
	  
    stage ('Deploy to server') {
            steps {
           sshagent(['application_server']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/DemoProject/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@3.86.253.216:/WebGoat'
		sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.1.2.60 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar &"'
              }      
           }     
    }
   
    stage ('Dynamic analysis') {
            steps {
           sshagent(['application_server']) {
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@52.90.65.32 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://3.86.253.216/WebGoat -x zap_report || true" '
		// sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.1.84.186 "sudo ./zap_report.sh"'
              }      
           }       
    }
  
    // stage ('Host vulnerability assessment') {
    //     steps {
    //          sh 'echo "In-Progress"'
    //         }
    // }

//    stage ('Security monitoring and misconfigurations') {
//         steps {
//              sh './securityhub.sh'
//             }
//     }
	
//    stage ('Incidents report') {
//         steps {
//           sh './final_report.sh'
//         }
//     }	  
	  
//    }  
// }
