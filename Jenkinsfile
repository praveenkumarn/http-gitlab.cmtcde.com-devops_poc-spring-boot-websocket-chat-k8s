#!/usr/bin/env groovy
 
/**
        * Sample Jenkinsfile for SpringBoot-Chat Application Pipeline
        * from https://github.com/praveenkumarn/spring-boot-websocket-chat-Kube/edit/master/Jenkinsfile
        * by Praveen
 */


timestamps {

node ('Kubernetes') {

	stage ('KGL_Complete_CI - Checkout') {
 	 checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '00a9b575-7866-4f8b-9995-6ea0281fa5b8', url: 'http://gitlab.cmtcde.com/root/spring-boot-websocket-chat-Kube.git']]]) 
	}
	stage ('KGL_Complete_CI - Build') {
 	
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

  
 // Docker Image build step
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
 """
  cleanWs()
		// Checkstyle report
		step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '90', pattern: '**/checkstyle-result.xml. ', unHealthy: '40']) 
	}
}
   stage("Quality Gate"){
    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
 }  
}
