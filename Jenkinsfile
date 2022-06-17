
pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'MAVEN_HOME'
    }

    stages {
        stage('Prepare-Workspace') {
            steps {
                // Get some code from a GitHub repository
                git credentialsId: 'github-server-credentials', url: 'https://github.com/venkat09docs/Maven-Java-Project.git'    
		stash 'Source'
            }
            
        }

	 stage('SonarQube analysis') {
         
          steps{
                echo "Sonar Scanner"
                  sh "mvn clean compile"
               withSonarQubeEnv('sonar-7') { 
                 sh "mvn sonar:sonar "
                }                     
          }
      }
	    
      stage('Unit Test Cases') {
         
          steps{
	       echo "Clean and Test"
              sh "mvn clean test"  
          }
          post{
              success{
		      echo "Clean and Test"
                  junit 'target/surefire-reports/*.xml'
              }
          }
      }
	    
      stage('Build Code') {
        
          steps{
	      unstash 'Source'
              sh "mvn clean package"  
          }
          post{
              success{
                  archiveArtifacts '**/*.war'
              }
          }
      }
	    
      stage('Build Docker Image') {
         
         steps{
                  sh "docker build -t gvenkat/webapp1 ."  
         }
     }
	    
     stage('Publish Docker Image') {
         
        steps{

    	      withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
    		    sh "docker login -u ${dockerUser} -p ${dockerPassword}"
	      }
        	sh "docker push gvenkat/webapp1"
         }
    }
	    
     stage('Deploy to Staging') {
	
	steps{
	      //Deploy to K8s Cluster 
              echo "Deploy to Staging Server"
	      sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	      sshCommand remote: kops, command: "kubectl delete -f Maven-Java-Project/k8s-code/staging/app/deploy-webapp.yml"
	      sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/staging/app/."
	}		    
    }
	    
     stage ('Integration-Test') {
	
	steps {
             echo "Run Integration Test Cases"
             unstash 'Source'
            sh "mvn clean verify"
        }
    }
	    
    stage ('approve') {
	steps {
		echo "Approval State"
                timeout(time: 7, unit: 'DAYS') {                    
			input message: 'Do you want to deploy?', submitter: 'admin'
		}
	}
     }
	    
     stage ('Prod-Deploy') {
	
	steps{
              echo "Deploy to Production"
	      //Deploy to Prod K8s Cluster
	      sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	      sshCommand remote: kops, command: "kubectl delete -f Maven-Java-Project/k8s-code/prod/app/deploy-webapp.yml"
	      sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/prod/app/."
	}
	}

    }
}
