pipeline {     
	agent any          
	
    tools {         
	    jdk 'jdk17'         
	    maven 'maven3' 
    	}      
    
    environment { 
        SCANNER_HOME = tool 'sonar-scanner' 
    } 

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Satyavankhandagale/Ekart.git'
            }
        }
   
    
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test') {
            steps { 
                sh "mvn test -DskipTests=true" 
            }
        }
    
        stage('SonarQube Analysis') {             
			steps { 
                withSonarQubeEnv('Sonar') { 
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART -Dsonar.java.binaries=. ''' 
                } 
            } 
        }  
        
        stage('Owas dependency check') {
            steps { 
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC' 
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps { 
                sh "mvn package -DskipTests=true" 
            } 
        }
        
        stage('Deploy Artifacts To Nexus') {             
			steps {  
			    withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
			    sh "mvn deploy -DskipTests=true" 
                } 
            } 
        }
        
        stage('Build & Tag Docker Image') {             
            steps {                 
                script { 
                  withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker'){
                    sh "docker build -t satyavan/ekart:latest -f docker/Dockerfile ."    
                    }
                } 
            } 
        }    
        stage('Trivy Scan Image') {             
			steps { 
                sh "trivy image satyavan/ekart:latest > trivy-report.txt " 
            } 
        }
        
        stage('Push Docker Image') {             
			steps {                 
			script { 
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') { 
                        sh "docker push satyavan/ekart:latest" 
                    } 
                } 
            } 
        } 

	stage('Deploy To K8s') {             
		steps { 
                	withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.15.105:6443') {
   	 		sh "kubectl apply -f deploymentservice.yml -n webapps"      sleep 30               
			sh "kubectl get svc -n webapps" 
                } 
            } 
        } 
	    
        
    } 
    
}
