#!groovy

import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import jenkins.util.*;
import jenkins.model.*;
import java.net.URL


node {
   properties([parameters([string(defaultValue: '', description: '', name: 'DEPLOY_ENV')]), pipelineTriggers([])])
   def GIT_URL = 'https://bitbucket.org/.git'
   def GIT_BRANCH = '*/develop'
   def MAVEN_GOALS = 'mvn clean install -X dependency:tree'
   def MAVEN_DEPLOY = "mvn -Dorg.arm.username=user-id -Dorg.arm.password=password -Denv=int package deploy"
   def MAVEN_DEPLOY1 = "mvn -Dorg.arm.username=user-id -Dorg.arm.password=password -Denv=$DEPLOY_ENV package deploy"
   def MAVEN_DEPLOY2 = "mvn -Dorg.arm.username=user-id -Dorg.arm.password=password -Denv=$DEPLOY_ENV package deploy"
   def MAVEN_DEPLOY3 = "mvn -Dorg.arm.username=user-id -Dorg.arm.password=password -Denv=$DEPLOY_ENV package deploy"
   
	/******************************
	*  Checkout code from GIT 
	***********************************/
	
		stage('\u2776 Git Checkout Code, Clean Install & Archive Artifact') 
		echo "INFO => Checking out from URL: ${GIT_URL} and BRANCH: ${GIT_BRANCH}"
		checkout([$class: 'GitSCM', branches: [[name: '*/develop']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', extensions: [[$class: 'CleanBeforeCheckout']], relativeTargetDir: 'D:\\workspace\\${JOB_NAME}']], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_URL}"]]])

     	/***********************
       * Build stage to trigger the maven build
   	**************************/
   
		echo "INFO => MAVEN_HOME: ${MAVEN_HOME}"
		echo "INFO => JAVA_HOME : ${JAVA_HOME}"
		bat "cd D:\\workspace\\${JOB_NAME} && ${MAVEN_GOALS}"
	
    	 /************************************************************
      	 * Archive Artifact
     	 ************************************************************/
 	
		archiveArtifacts '**\\*.zip'
		fingerprint '**\\*.zip'
		bat 'xcopy /i D:\\workspace\\%JOB_NAME%\\target\\*-1.0.0.zip D:\\ArchivedArtifact\\%JOB_NAME%-%BUILD_NUMBER%.zip'
		
        /***************************************************
       * maven deploy on target env
        ****************************************************/
		switch("$DEPLOY_ENV") {
      		 case "sit":
      		 echo "$DEPLOY_ENV"
	   		stage("\u2777 Deploy $DEPLOY_ENV")
	
		echo "INFO => MAVEN_HOME: ${MAVEN_HOME}"
		echo "INFO => JAVA_HOME : ${JAVA_HOME}"
		bat "cd D:\\workspace\\${JOB_NAME} && ${MAVEN_DEPLOY1}"	
	/**********
       * Jmeter & Pulblish Xunit Report
        ***********/
		sleep(120)
		stage ("\u2777 JMeter Smoke Test $DEPLOY_ENV")

       	 	bat 'C:\\apache-jmeter-3.1\\bin\\jmeter.bat -Jjmeter.save.saveservice.output_format=xml -n -t D:\\workspace\\%JOB_NAME%\\src\\test\\jmeter\\functional-tests.jmx -l testResult.jtl -Japi.host=api-prd.org.com'
   		step([$class: 'XUnitPublisher', testTimeMargin: '4000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'], [$class: 'SkippedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0']], tools: [[$class: 'CustomType', customXSL: 'D:\\workspace\\MULESOFT\\jmeter-to-xunit.xsl', deleteOutputFiles: true, failIfNotNew: false, pattern: '**\\testResult.jtl', skipNoTestFiles: true, stopProcessingIfError: false]]])
      		 break
	  	 case "prf":
      		 echo "$DEPLOY_ENV"
	   	stage("\u2777 Deploy $DEPLOY_ENV")
	
		echo "INFO => MAVEN_HOME: ${MAVEN_HOME}"
		echo "INFO => JAVA_HOME : ${JAVA_HOME}"
		bat "cd D:\\workspace\\${JOB_NAME} && ${MAVEN_DEPLOY2}"
		
	/**********
        * Jmeter & Pulblish Xunit Report
        ***********/
		sleep(120)
		stage ("\u2777 JMeter Smoke Test $DEPLOY_ENV")

       		bat 'C:\\apache-jmeter-3.1\\bin\\jmeter.bat -Jjmeter.save.saveservice.output_format=xml -n -t D:\\workspace\\%JOB_NAME%\\src\\test\\jmeter\\functional-tests.jmx -l testResult.jtl -Japi.host=api-sit.org.com'
   		step([$class: 'XUnitPublisher', testTimeMargin: '4000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'], [$class: 'SkippedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0']], tools: [[$class: 'CustomType', customXSL: 'D:\\workspace\\MULESOFT\\jmeter-to-xunit.xsl', deleteOutputFiles: true, failIfNotNew: false, pattern: '**\\testResult.jtl', skipNoTestFiles: true, stopProcessingIfError: false]]])
       		break
	  	 case "prd":
      		 echo "$DEPLOY_ENV"
	  	 stage("\u2777 Deploy $DEPLOY_ENV")
	
		echo "INFO => MAVEN_HOME: ${MAVEN_HOME}"
		echo "INFO => JAVA_HOME : ${JAVA_HOME}"
		bat "cd D:\\workspace\\${JOB_NAME} && ${MAVEN_DEPLOY3}"
	
	/**********
      	 * Jmeter & Pulblish Xunit Report
        ***********/
		sleep(120)
		stage ("\u2777 JMeter Smoke Test $DEPLOY_ENV")

		bat 'C:\\apache-jmeter-3.1\\bin\\jmeter.bat -Jjmeter.save.saveservice.output_format=xml -n -t D:\\workspace\\%JOB_NAME%\\src\\test\\jmeter\\functional-tests.jmx -l testResult.jtl -Japi.host=api-prf.org.com'
		step([$class: 'XUnitPublisher', testTimeMargin: '4000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'], [$class: 'SkippedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0']], tools: [[$class: 'CustomType', customXSL: 'D:\\workspace\\MULESOFT\\jmeter-to-xunit.xsl', deleteOutputFiles: true, failIfNotNew: false, pattern: '**\\testResult.jtl', skipNoTestFiles: true, stopProcessingIfError: false]]])
      		 break
	   	default:
	 	stage("\u2777 Deploy on int")
		
			
		echo "INFO => MAVEN_HOME: ${MAVEN_HOME}"
		echo "INFO => JAVA_HOME : ${JAVA_HOME}"
		bat "cd D:\\workspace\\${JOB_NAME} && ${MAVEN_DEPLOY}"
    
	/**********
       * Jmeter & Publish Xunit Report
        ***********/
		sleep(120)
		stage ("\u2777 JMeter Smoke Test int")
		bat 'C:\\apache-jmeter-3.1\\bin\\jmeter.bat -Jjmeter.save.saveservice.output_format=xml -n -t D:\\workspace\\%JOB_NAME%\\src\\test\\jmeter\\functional-tests.jmx -l testResult.jtl -Japi.host=api-int.org.com'
		step([$class: 'XUnitPublisher', testTimeMargin: '3000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'], [$class: 'SkippedThreshold', failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0']], tools: [[$class: 'CustomType', customXSL: 'D:\\workspace\\MULESOFT\\jmeter-to-xunit.xsl', deleteOutputFiles: true, failIfNotNew: false, pattern: '**\\testResult.jtl', skipNoTestFiles: false, stopProcessingIfError: false]]])
		break
    
   }

}
