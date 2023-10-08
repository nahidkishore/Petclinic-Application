pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage("Clean Workspace"){
            steps{
                cleanWs()
            }
        }


        stage("Git Checkout"){
            steps{
               git branch: 'main', url: 'https://github.com/nahidkishore/Petclinic-Application.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile -DskipTests=true"
            }
        }
          stage("Test Cases"){
            steps{
                sh "mvn test -DskipTests=true"
            }
        }
         stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        
        stage("SonarQube Quality Gate"){
       
             steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
         }

     }
        stage("Build Artifact"){
            steps{
                sh "mvn clean install -DskipTests=true"
            }
        }
        stage("OWASP Dependency Check"){
           steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage("Buuld and Push to Docker Hub"){
               steps{
                   
                echo 'login into docker hub and pushing image....'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     sh "docker build . -t petclinic-application"
                     sh "docker tag petclinic-application ${env.dockerHubUser}/petclinic-application:latest"
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push ${env.dockerHubUser}/petclinic-application:latest"


               }
           }
         }

           stage("TRIVY"){
            steps{

                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     
                    sh "trivy image ${env.dockerHubUser}/petclinic-application:latest > trivy.txt" 


               }
                
            }
        }
         
          stage("Deploy to Container"){
            steps{
                sh " docker run -d --name petclinic-application -p 8082:8082 nahid0002/petclinic-application:latest "
            }
        }
        
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/Devops-CICD-Real-Time-Projects/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }
         
          stage('Deploy to kubernets') {
            steps {
                script {
                   
                    withKubeConfig([credentialsId: 'K8s', serverUrl: '']) {
                        sh ('kubectl apply -f deployment.yaml')
                    }
                }
            }
        }

        
       
        
        
       
        
      




     
    }
}
