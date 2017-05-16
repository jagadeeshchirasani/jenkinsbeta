#!groovy
/**********************************************************************
***** Description :: This template is used to Appx CI/CD pipeline *****
***** Author      :: Mukul Garg                                   *****
***** Date        :: 02/24/2017                                   *****
***** Revision    :: 2.0                                          *****
***********************************************************************/
import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL


def GIT_BRANCH = "master"
def GIT_REPO = "https://github.com/jagadeeshchirasani/jenkinsbeta.git"


properties (
  [buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '2')), 
  [$class: 'OptionalJobProperty', autoRebuild: false, rebuildDisabled: false], 
  parameters (
    [[$class: 'JobProperty', branch: '', branchFilter: '.*', defaultValue: "${GIT_BRANCH}", description: 'Select the Branch', name: 'Branch', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH', useRepository: "${GIT_REPO}"],
//    choice(choices: ['DEV', 'PROD'], description: 'Please select Environment to deploy code.', name: 'ENV'),
	booleanParam(defaultValue: true, description: 'Clone GIT Repo. Default true', name: 'Checkout'), booleanParam(defaultValue: false, description: 'Build the Code', name: 'npm_build'), booleanParam(defaultValue: false, description: 'Deploy the artifact to DEV/PROD environment', name: 'Deploy')]
  ), 
	pipelineTriggers([])]
)


/********************
** Email Properties**
********************/
RECIPIENTS = 'jchir2@sapient.com'
/********************/

def notify(String recipients) {
  emailext (
      subject: "[${currentBuild.result}]: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>[${currentBuild.result}]: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
	  to: "${recipients}",
      recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']]
    )
}

/********************
**** Clone Code *****
********************/

def checkout(String branch,String repo) {
  checkout (
      [$class: 'GitSCM', branches: [[name: "${branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], 
	  userRemoteConfigs: [[credentialsId: 'git', url: "${repo}"]]]
	)
}

/********************************
**** Build Code Using Maven *****
********************************/

def buildCode() {
   sh '''npm install && command || exit 19
         npm run build:webpack && command || exit 19'''
}

/****************************************
**** Deploy Artifact to Remote Server ***
****************************************/

def deployArtifact() {
choice = new ChoiceParameterDefinition('Environment', ['DEV', 'PROD'] as String[], 'Description')
def max = input(message: 'Select Environment', parameters: [choice])
sh """#!/bin/bash
tar -czvf dist-${env.BUILD_NUMBER}.tar.gz dist
"""

sh """#!/bin/bash
   case ${max} in
    DEV)
	  ####### Taking backup of old files ########
      echo "Taking backup"
      today=`date '+%Y_%m_%d__%H_%M_%S'`
      tar -czvf /app/appx_backup/appx_html-\$today.tar.gz /app/appx_html/
      echo "Removing dist packages"
      rm -rf /app/appx_html/css /app/appx_html/fonts /app/appx_html/images /app/appx_html/index.html /app/appx_html/js
      echo "Deploying new packages"
      cp -pr dist/* /app/appx_html/
     ;;
    PROD)
     ;;
esac
"""
}

node('master'){
	
	if (Checkout == "true") {
	    stage('Clone Code') {
	        echo "Clone Code from ${GIT_REPO}"
		    checkout(Branch,GIT_REPO)
	    }
	}
	if (npm_build == "true") {
	    stage('Build Code') {
	        echo "Building the code using Maven"
		    buildCode()
	    }
	}
	if (Deploy == "true") {
	    stage('Deploy File') {
		   echo "Deploy Tar File"
		   deployArtifact()
		}
	}
	notify(RECIPIENTS)
}