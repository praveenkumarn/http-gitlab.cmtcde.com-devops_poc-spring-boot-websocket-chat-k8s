#!/usr/bin/env groovy
 
/**
        * Sample Jenkinsfile for SpringBoot-Chat Application Pipeline
        * from https://github.com/praveenkumarn/spring-boot-websocket-chat-k8s/edit/master/Jenkinsfile
        * by Praveen
 */


timestamps {

node ('Kubernetes') {

	stage ('K8sGL_CICD- Checkout') {
 	 checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '00a9b575-7866-4f8b-9995-6ea0281fa5b8', url: 'http://gitlab.cmtcde.com/devops_poc/spring-boot-websocket-chat-k8s.git']]]) 
	}
	stage ('K8sGL_CICD - Build') {
 	
	withMaven(maven: 'M2_HOME') { 
 			if(isUnix()) {
 				sh "mvn -f pom.xml clean verify org.apache.maven.plugins:maven-jxr-plugin:2.5:jxr org.apache.maven.plugins:maven-pmd-plugin:3.10.0:pmd org.apache.maven.plugins:maven-pmd-plugin:3.10.0:cpd org.apache.maven.plugins:maven-checkstyle-plugin:3.0.0:checkstyle org.codehaus.mojo:findbugs-maven-plugin:3.0.1:findbugs"
                sh "mvn -f pom.xml test org.codehaus.mojo:cobertura-maven-plugin:2.7:cobertura -Dcobertura.report.format=xml"
                sh "mvn -f pom.xml package " 
			} else { 
 				bat "mvn -f pom.xml clean verify org.apache.maven.plugins:maven-jxr-plugin:2.5:jxr org.apache.maven.plugins:maven-pmd-plugin:3.10.0:pmd org.apache.maven.plugins:maven-pmd-plugin:3.10.0:cpd org.apache.maven.plugins:maven-checkstyle-plugin:3.0.0:checkstyle org.codehaus.mojo:findbugs-maven-plugin:3.0.1:findbugs"
                bat "mvn -f pom.xml test org.codehaus.mojo:cobertura-maven-plugin:2.7:cobertura -Dcobertura.report.format=xml"
                bat "mvn -f pom.xml package " 
			} 
 		}		
 	
  stage('SonarQube analysis') {
     def scannerHome = tool 'SonarScanner';
     withSonarQubeEnv('SonarQube') { 
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }



// Shell Pre-build step
 		
sh label: '', script: '''
process_count=`kubectl get services | grep kubernetes-springboot | grep -v grep | wc -l`
if [ "${process_count}" -eq "0" ] ; then
     echo "kubernetes-springboot not running.No action required"
else
     echo "kubernetes-springboot services running, so delete the services and deployment"
    	kubectl delete services kubernetes-springboot
        kubectl delete -n default deployment kubernetes-springboot
fi
'''


// Docker Image build and deploy step
sh """ 
#!/bin/bash
pwd
id
ls -lrt
java -version

docker image ls
docker container ls

docker build -t spring-boot-websocket-chat-demo .

docker tag spring-boot-websocket-chat-demo praveenkumarnagarajan/spring-boot-websocket-chat-demo:0.0.1-SNAPSHOT

cat ~/pass.txt | docker login --username praveenkumarnagarajan --password-stdin

docker push praveenkumarnagarajan/spring-boot-websocket-chat-demo:0.0.1-SNAPSHOT 
docker pull praveenkumarnagarajan/spring-boot-websocket-chat-demo:0.0.1-SNAPSHOT

docker image ls
kubectl run kubernetes-springboot --image=praveenkumarnagarajan/spring-boot-websocket-chat-demo:0.0.1-SNAPSHOT --port=8080
kubectl expose deployment/kubernetes-springboot --type="NodePort" --port 8080

kubectl get nodes
kubectl get services
kubectl describe services/kubernetes-springboot
 """
  cleanWs()
		// Checkstyle report
		step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '90', pattern: '**/checkstyle-result.xml. ', unHealthy: '40']) 
	}
}
     stage("Quality Gate"){
    timeout(time: 2, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
 }  
}

   cleanWs()

