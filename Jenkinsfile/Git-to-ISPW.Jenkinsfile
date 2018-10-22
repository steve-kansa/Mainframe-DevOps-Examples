node{
    stage("create file"){
        touch file: 'example', timestamp: 0
        bat "@echo version := 1.0.${env.BUILD_ID} >> example"
        
        //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'steve-kansa-github', url: 'https://github.com/steve-kansa/SXK1.git']]])

    }

    stage("Pull from git"){
        
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'steve-kansa-github', url: 'https://github.com/steve-kansa/SXK1.git']]])

        def Cobolfiles = findFiles(glob: 'MF_Source/*.cbl')
        def Copybooks  = findFiles(glob: 'MF_Source/*.cpy')
        def JCL        = findFiles(glob: 'MF_Source/*.jcl')

        Cobolfiles.each
        {
            echo "Copying ${it.path}"
            bat "xcopy /s /Y " + it.path + " gitsource\\MF_Source"
            echo "Renaming ${it.path}"
            bat "ren " +it.path + it.name + " " + it.name.trim().split("\\.")[0]
        }
        Copybooks.each
        {
            echo "Copying ${it.path}"
            bat "xcopy /s /Y " + it.path + " gitsource\\MF_Source"
            echo "Renaming ${it.path}"
            bat "ren " +it.path + it.name + " " + it.name.trim().split("\\.")[0]
        }
        JCL.each
        {
            echo "Copying ${it.path}"
            bat "xcopy /s /Y " + it.path + " gitsource\\MF_Source"
            echo "Renaming ${it.path}"
            bat "ren " +it.path + it.name + " " + it.name.trim().split("\\.")[0]
        }
        /*dir("gitsource")
        {
            stdout = bat(returnStdout: true, script: "git add .")
            echo stdout

            message = "Changes for SetID " + SetId + " from User: " + Owner
        
            stdout = bat(returnStdout: true, script: "git config --global user.name ${Owner}")
            echo stdout

            Owner = Owner + "@compuware.com"
            stdout = bat(returnStdout: true, script: "git config --global user.email ${Owner}")
            echo stdout

            stdout = commitChangeset = bat(returnStdout: true, script: 'git diff')
            echo commitChangeset

            shortCommit = bat(returnStdout: true, script: "git log -n 1 -p)
            echo shortCommit

            stdout = bat(returnStdout: true, script: "git commit -m \"${message}")
            echo stdout

            stdout = bat(returnStdout: true, script: "git tag -a ${SetId} -m \"Changes for SetId")
            echo stdout    

            withCredentials([usernamePassword(credentialsId: 'steve-kansa-github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                stdout = bat(returnStdout: true, script: "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/steve-kansa/SXK1.git HEAD:master --tags")
                echo stdout
            }
        }*/
    }



    stage("FTP"){
        echo "FTP TO CWCC"
        
        ftpPublisher alwaysPublishFromMaster: false, 
        continueOnError: false, 
        failOnError: false, 
        publishers: [[configName: 'cwcc', 
        transfers: [[asciiMode: true, 
        cleanRemote: false, 
        excludes: '', 
        flatten: false, 
        makeEmptyDirs: false, 
        noDefaultExcludes: false, 
        patternSeparator: '[, ]+', 
        remoteDirectory: '//BATCH.JCL/', 
        remoteDirectorySDF: false, 
        removePrefix: '', 
        sourceFiles: '*']], 
        usePromotionTimestamp: false, 
        useWorkspaceInPromotion: false, 
        verbose: true]]

    }
}