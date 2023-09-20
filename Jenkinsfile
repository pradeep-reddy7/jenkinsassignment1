pipeline{
    agent any
    
    stages{
        stage("Git Checkout"){
            steps{
                git credentialsId: 'assignment1', url: 'https://github.com/pradeep-reddy7/jenkinsassignment1'
            }
        }

        stage('Quality Gate Status Check'){
            steps{
                script{
			withSonarQubeEnv(credentialsId: 'sonar-token') { 
                        // Get Home Path of Maven - This is a comment
                        def mvnHome = tool name: 'maven-3', type: 'maven'
			sh "${mvnHome}/bin/mvn clean sonar:sonar"
                       	  }
			            timeout(time: 20, unit: 'SECONDS') {
			            def qg = waitForQualityGate()
				                if (qg.status != 'OK') {
					                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
				                }
                          }
                }
            }  
        }
        stage("Maven Build"){
            steps{
                script{
                // Get Home Path of Maven 
                def mvnHome = tool name: 'maven-3', type: 'maven'
                sh "${mvnHome}/bin/mvn package"
                }
            }
        }
	
	
	stage("Upload War To Nexus"){
	    steps{
		script{
		    def mavenPom = readMavenPom file: 'pom.xml'
		    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "assignment1-SNAPSHOT" : "assignment1-release"
		    nexusArtifactUploader artifacts: [
			[
			    artifactId: 'assignment1', 
		            classifier: '', 
			    file: "target/assignment1-${mavenPom.version}.war", 
			    type: 'war'
			]
			], 
			    credentialsId: 'nexus3', 
			    groupId: 'in.javahome', 
			    nexusUrl: '172.31.1.97:8081', 
			    nexusVersion: 'nexus3', 
			    protocol: 'http', 
			    repository: nexusRepoName, 
			    version: "${mavenPom.version}"    
                       }
		}
	}
	
        stage("Deploy to Tomcat Server"){
            steps{
                sshagent(['tomcat-keypair']) {
                sh """
		    echo $WORKSPACE
		    mv target/*.war target/assignment1.war
                    scp -o StrictHostKeyChecking=no target/assignment1.war  ec2-user@172.31.42.77:/opt/tomcat8/webapps/
                    ssh ec2-user@172.31.42.77 /opt/tomcat8/bin/shutdown.sh
                    ssh ec2-user@172.31.42.77 /opt/tomcat8/bin/startup.sh
                
                """
                }
            
            }
        }
    } 
    
     //post {
	  //always {
 	    //echo 'Deleting the Workspace'
 	    //deleteDir() /* Clean Up our Workspace */
 	  //}
 	    //success {
 		//mail to: 'opensourcedevopstraining94@gmail.com',
 		  //subject: "Success Build Pipeline: ${currentBuild.fullDisplayName}",
 		  //body: "The pipeline ${env.BUILD_URL} completed successfully"
 	    //}
 	    //failure {
   	        //mail to: 'opensourcedevopstraining94@gmail.com',
  		  //subject: "Failed Build Pipeline: ${currentBuild.fullDisplayName}",
  		  //body: "Something is wrong with ${env.BUILD_URL}"
  	    //}
     //}
}