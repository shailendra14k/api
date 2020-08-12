def appName="shailendra"
def mvnCmd="mvn -s configuration/settings.xml"
def templatePath="cicd/template.json"
def sonarHost="http://sonarqube-nexus1.apps.shsingh.5601.sandbox567.opentlc.com"
def PROJECT="shailendra"
pipeline{
		agent{
				label 'maven'
		}
		stages{
				stage('Checkout'){
						steps{
								checkout scm
						}
				}
				stage('Code Compile'){
						steps{
								sh "${mvnCmd} clean package -DskipTests=true -Dversion=${env.BUILD_NUMBER}"
						}
				}
				/*stage('Execute Test Cases'){
						steps{
								sh "${mvnCmd} test -Dversion=${env.BUILD_NUMBER}"
								junit "target/surefire-reports/*.xml"
						}
				}
				stage('Sonar Analysis'){
						steps{
								sh "${mvnCmd} sonar:sonar -Dsonar.host.url=${sonarHost} -DskipTests=true -Dversion=${env.BUILD_NUMBER}"
						} 
				}*/
				stage('Upload artifact to nexus'){
						steps{
								sh "${mvnCmd} deploy -DskipTests=true -Dversion=${env.BUILD_NUMBER}"
						}
				}
				stage('Deploy Template Dev'){
						steps{
								script{
										try{
												openshift.withCluster(){
														openshift.withProject("${env.PROJECT}"){
																echo "Using project: ${openshift.project()}"
																echo "${env.PROJECT}"
																echo "${appName}"
																if(!openshift.selector("svc",[template:"${appName}"]).exists() || !openshift.selector("dc",[template:"${appName}"]).exists() || !openshift.selector("route",[template:"${appName}"]).exists()){
																        openshift.newApp(templatePath,"-p PROJECT=${env.PROJECT}")
																}
																
																}
														}
												}
										 catch(e){
										        print e.getMessage()
												echo "This stage has an exception that can be ignored."
										}
								}
						}
				}
				stage('Image build'){
						steps{
								script {
										try{
												timeout(time: 120, unit: 'SECONDS') {
														openshift.withCluster(){
																openshift.withProject(env.PROJECT){
																		echo "Using project: ${openshift.project()}"
																		def build = openshift.selector("bc","${appName}").startBuild("--from-file=target/api1-1.0-SNAPSHOT.jar", "--wait=true")
																		build.untilEach{
																				return it.object().status.phase == "Complete"
																		}
																}
														}
												}
										} catch(e) {
												print e.getMessage()
												error "Build not successful"
										}
								}
						}
				}
				stage('Image Tag'){
						steps{
								script{
										openshift.withCluster(){
												openshift.tag("${PROJECT}/${appName}:latest", "${PROJECT}/${appName}:${env.BUILD_NUMBER}")
										}
								}
						}
				}
				
				
		}

}
