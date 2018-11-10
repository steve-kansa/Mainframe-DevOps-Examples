#!/usr/bin/env groovy
import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurper
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

 String Git_Credentials      = "github"
 //String Git_URL              = "https://github.com/${Git_Project}"
 String Git_TTT_Repo         = "${ISPW_Stream}_${ISPW_Application}_Unit_Tests.git"
 String Git_Branch           = "master"
 String SQ_Scanner_Name      = "scanner" 
 String SQ_Server_Name       = "localhost"  
 String MF_Source            = "MF_Source"
 String XLR_Template         = "A Release from Jenkins"
 String XLR_User	         = "admin"	
		 
/**
 Helper Methods for the Pipeline Script
*/
 // Add whichever params you think you'd most want to have
// replace the slackURL below with the hook url provided by
// slack when you configure the webhook


def notifySlack(text, channel) {
    def slackURL = 'https://hooks.slack.com/services/xxxxxxx/yyyyyyyy/zzzzzzzzzz'
    def payload = JsonOutput.toJson([text      : text,
                        channel   : channel,
                        username  : "jenkins",
                        icon_emoji: ":jenkins:"])
    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}
def runCommand(cmd, workspacedir) 
{
    def exists = fileExists "${workspacedir}\\output.txt"

    if (exists) {
        bat(returnStatus:true , script: "del ${workspacedir}\\output.txt")
    } 
    
    cmd = cmd + " > ${workspacedir}\\output.txt"
       
    if (isUnix()){
         return sh(returnStatus:true , script: '#!/bin/sh -e\n' + cmd).trim()
     } else{
       status = bat(returnStatus:true , script: cmd)
       //echo "Command Returned Status: " + status
       output = readFile "${workspacedir}\\output.txt"
       //echo "Command Output: " + output
       return status
    } 
}
 /**
 Wrapper around the Git Plugin's Checkout Method
 @param URL - URL for the git server
 @param Branch - The branch that will be checked out of git
 @param Credentials - Jenkins credentials for logging into git
 @param Folder - Folder relative to the workspace that git will check out files into
*/
 def gitcheckout(String URL, String Branch, String Credentials, String Folder)
 {
        println "Scenario " + URL
        println "Scenario " + Branch
        println "Scenario " + Credentials
        checkout changelog: false, poll: false, 
        scm: [$class: 'GitSCM', 
        branches: [[name: "*/${Branch}"]], 
        doGenerateSubmoduleConfigurations: false, 
        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "${Folder}"], 
        [$class: 'LocalBranch', localBranch: 'local']], 
        submoduleCfg: [], 
        userRemoteConfigs: [[credentialsId: "${Credentials}", name: 'origin', url: "${URL}"]]]
 }
 /* 
  Node is a required part of the Jenkins pipeline for any steps to be executed
*/ 
node{
    /* 
     This stage is used to retieve source from ISPW
     */ 
    stage("Retrieve Code From ISPW")
    {
            //Retrieve the code from ISPW that has been promoted 
            checkout([$class: 'IspwContainerConfiguration', 
            componentType: '',                  // optional filter for component types in ISPW
            connectionId: "${HCI_Conn_ID}",     
            credentialsId: "${HCI_Token}",      
            containerName: "${SetId}",   
            containerType: '2',                 // 0-Assignment 1-Release 2-Set
            ispwDownloadAll: false,             // false will not download files that exist in the workspace and haven't previous changed
            serverConfig: '',                   // ISPW runtime config.  if blank ISPW will use the default runtime config
            serverLevel: ''])                   // level to download the components from
    }
     /* 
     This stage pushes the changed code from ISPW into a GitHub repo  
     */ 
    stage("Commit to git"){

        workspacedir = "${WORKSPACE}"

        Git_URL = "https://github.com/steve-kansa/SXK1"
        gitcheckout(Git_URL, Git_Branch, Git_Credentials, "gitsource")
        
        def Cobolfiles = findFiles(glob: 'SXK1/MF_Source/*.cbl')
        def Copybooks  = findFiles(glob: 'SXK1/MF_Source/*.cpy')
        
        Cobolfiles.each{
            echo "Copying ${it.path}"
            cmd = "xcopy /s /Y " + it.path + " gitsource\\MF_Source"
            runCommand(cmd, workspacedir)
        }
        Copybooks.each{
            echo "Copying ${it.path}"
            cmd "xcopy /s /Y " + it.path + " gitsource\\MF_Source"
            runCommand(cmd, workspacedir)
        }
        
        dir("gitsource"){

            runCommand("git config --global user.name ${Owner}", workspacedir)
            Owner = Owner + "@compuware.com"
            runCommand("git config --global user.email ${Owner}", workspacedir)
            runCommand("git add .", workspacedir)
            
            message = "Changes for SetID: " + SetId + " from User: " + Owner

            result = runCommand("git commit -m \"${message}\"", workspacedir)


            modifiedFiles = sh(returnStdout: true, script: "#!/bin/sh -e\n git log -n 1 --name-status | grep -E ^M[[:space:]] | sort | uniq")
            deletedFiles = sh(returnStdout: true, script: "#!/bin/sh -e\n git log -n 1 --name-status | grep -E ^D[[:space:]] | sort | uniq")
            addedFiles = sh(returnStdout: true, script: "#!/bin/sh -e\n git log -n 1  --name-status | grep -E ^A[[:space:]] | sort | uniq")
            renamedFiles = sh(returnStdout: true, script: "#!/bin/sh -e\n git log -n 1 --name-status | grep -E ^R[[:space:]] | sort | uniq")
            copiedFiles = sh(returnStdout: true, script: "#!/bin/sh -e\n git log -n 1 --name-status | grep -E ^C[[:space:]] | sort | uniq")

            buf = modifiedFiles.split('\n')

            buf.each{
                file = it.substring(2)
                filename = file.split("/")
                member = filename[1].split("\\.")
                membertype = member[0]
                membername = member[1]
                echo "Member Type->${membertype}Member->${membername}\nFile->${file}\n"
            }
            
            
            if(result > 0){   
                output = readFile "${workspacedir}\\output.txt"
                if (output.contains("nothing to commit")){
                    currentBuild.result = 'SUCCESS' // Set build to Success
                    return
                }
            }

            runCommand('git diff', workspacedir)
            
            // Check to see if git detected changes, if no changes are detected, nothing further to do in this step, exit stage
            result = runCommand("git tag -a ${SetId} -m \"Changes for SetId ${SetId}\"", workspacedir)  
            if(result > 0){
                output = readFile "${workspacedir}\\output.txt"
                if (output.contains("no changes")){
                    currentBuild.result = 'SUCCESS' // Set build to Success
                    echo "tag already exists" 
                    return
                }
            }
             withCredentials([usernamePassword(credentialsId: 'stevekansagithub', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                result = runCommand("git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/steve-kansa/SXK1.git HEAD:master --tags", workspacedir)
                if(result > 0){
                    echo "Push to Git Server failed!!!!!!!"
                    currentBuild.result = 'FAILURE' // Set build to Success
                }
                
            }
        }
    }
} 
