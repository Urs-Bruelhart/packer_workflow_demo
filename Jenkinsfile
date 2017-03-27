/***********************************************************************
                         Aricent Technologies Proprietary
 
This source code is the sole property of Aricent Technologies. Any form of utilization
of this source code in whole or in part is  prohibited without  written consent from
Aricent Technologies
 
File Name			        : Jenkinsfile
Principal Author	    : PRAVEEN KUMAR KRISHNAMOORTHY
Subsystem Name        : Packer Workflow Code
Module Name           :
Date of First Release : Mar 27, 2017
Author                : PRAVEEN KUMAR KRISHNAMOORTHY
Description           : This is a sample program to study Continous Integration
Version               : 1.0
Date(DD/MM/YYYY)      : Mar 27, 2017
Modified by           : PRAVEEN KUMAR KRISHNAMOORTHY
Description of change : Forked From Existing Pipeline Script
 
***********************************************************************/
node {
    def passstr = scmPassword
    //To escape all Special Charecters in a given input string password
    passstr = passstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
    scmPassword = passstr.replaceAll( /([@])/, '%40' )
  
    def userstr = scmUsername
    //To escape all Special Charecters in a given input string Username
    userstr = userstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
    scmUsername = userstr.replaceAll( /([@])/, '%40' )
  
  
    stage('Code Pickup') {
    echo "Source Code Repository Type : ${CodeType}"
    echo "Source Code Repository Path : ${CodeLoc}"
    
    if("${CodeType}".toUpperCase()=='SVN'){
        sh "svn co --username ${scmUsername} --password ${scmPassword} ${CodeLoc} ."
        
    } else if("${CodeType}".toUpperCase()=='GIT'){
        CodeLoc = CodeLoc.substring(0, CodeLoc.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +CodeLoc.substring(CodeLoc.indexOf("//")+2, CodeLoc.length());
        try {
            sh 'ls -a | xargs rm -fr' 
        } catch (error) {
        }
        sh "git clone ${CodeLoc} ."        
    } else {
        error 'Unknown Source code repository. Only GIT and SVN are supported'
    }
  } 
//END OF CODE PICKUP STAGE
//_______________________________________________________________________________________________________________________________________________________________________  
//BUILD & PACKAGE
def appModuleSeperated = fileExists 'app'
def testModuleSeperated = fileExists 'test'
def appPath = ''
def testPath = ''  
if (appModuleSeperated) {
    echo 'App Module is found , assumed that application is present in /app directory'
    appPath='app/'
} else {
    echo 'There is no defined Application path , hence it is assumed that application is in current directory'
    appPath = ''
}

if (testModuleSeperated) {
    echo 'Test Module is found , assumed that Test Cases are Present for the concerned Modules and has to be performed'
    testPath = 'test/'
} else {
    echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
    testPath = ''
}
  if (appPath + fileExists("${FileName}")) {
    echo 'This application contains a single packer file , it is assumed to be developed in compiler in-dependant / interpreter based programming language.'
    distPackerFile = appPath + "${FileName}"
} else {
    echo 'Packerfile not found under ' + appPath
  }
// COPYING APP Directory to Current Working Directory
  def appWorkingDir = (appPath=='') ? '.' : appPath.substring(0, appPath.length()-1)  
// NEXUS file for Time Stamp comparison. This file is used for comparing time stamps and differentiating input files from generated output files.        
    sh 'echo Nexus>Nexus.txt'
//END OF INITIALIZING.
//_______________________________________________________________________________________________________________________________________________________________________  
//BUILD & PACKING
  //---------------------------------------
  if("${stage}".toUpperCase() == 'VALIDATE') {
    echo "Running packer validate on : ${distPackerfile}"
    sh "packer -v ; packer validate ${distPackerfile}"
  
  }
  if("${stage}".toUpperCase() == 'BUILD') {
    echo 'It is inferred that the package is a Build application , hence it has to be validated , built and moved to a temporary repository'
      stage('BUILD') {
        echo "Running packer validate on : ${distPackerfile}"
        sh "packer -v ; packer validate ${distPackerfile}"
        sh "packer build ${distPackerfile}"
    }   
  }  else if ("${stage}".toUpperCase() == 'TEST'){
    echo 'It is inferred that the package is a test application , hence it has to be moved to a provisioned with a runtime sandbox environment , validate , build and tested before pushing into repo'
        stage('TEST') {
        echo "Running packer validate on : ${distPackerfile}"
        sh "packer -v ; packer validate ${distPackerfile}"
        sh "packer build ${distPackerfile}"
      }   
  }

//END OF IMAGE PUSHING INTO REPOSITORY
// NEXUS UPDATE
  stage('Publish Jenkins Output to Nexus'){
        echo 'Publishing the artifacts...';
        sh 'rm Nexus.txt'
//NEXUS FLOW ENDS HERE
  }
}
