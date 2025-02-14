def status = [
    SUCCESS: "good",
    FAILURE: "danger",
    UNSTABLE: "warning",
    UNKNOWN: "warning",
    ABORTED: "warning",
    PREVSUCCESS: "good",
    PREVFAILURE: "warning",
    PREVUNSTABLE: "good",
    PREVUNKNOWN: "warning",
    PREVABORTED: "warning",
]
def emojis = [
    SUCCESS: "\\uD83D\\uDE0E",        // Smiling face with sunglasses
    FAILURE: "\\uD83D\\uDD25",        // Fire
    UNSTABLE: "\\uD83D\\uDE15",       // Confused face
    UNKNOWN: "\\uD83E\\uDD2A",        // Crazy face
    ABORTED: "\\uD83E\\uDD28",        // Face with raised eyebrow
    PREVSUCCESS: "\\uD83D\\uDE0E",    // Smiling face with sunglasses
    PREVFAILURE: "\\uD83D\\uDCA3",    // Bomb
    PREVUNSTABLE: "\\uD83D\\uDE15",   // Confused face
    PREVUNKNOWN: "\\uD83E\\uDD2A",    // Crazy face
    PREVABORTED: "\\uD83E\\uDD28",    // Face with raised eyebrow
]

//This is for if you want the delete and create of branches to not fail or run a build
if( COMMIT == '0000000000000000000000000000000000000000' ) {
   currentBuild.result = 'SUCCESS'
   currentBuild.displayName = "Deleted ${BRANCH} by ${env.BY}"
   return 
}         
if( COMMIT != '0000000000000000000000000000000000000000' && COMMITS == '[]' ) {
   currentBuild.result = 'SUCCESS'
   currentBuild.displayName = "Created ${BRANCH} by ${env.BY}"
   return 
} 

//Pipeline
pipeline {
    agent { label 'CMA-AZMEDTECH-BUILD' }
    options {
        durabilityHint 'PERFORMANCE_OPTIMIZED'
        timestamps()
        // set below as appropriate for project activity
        buildDiscarder logRotator(daysToKeepStr: '20', numToKeepStr: '50')
    }
    environment {
        PREVIOUS_BUILD_STATUS = "${status['PREV' + currentBuild.previousBuild?.currentResult ?: 'UNKNOWN']}"
        MAIL_GROUP = "AZMEDICAREIT@Cigna.com"
        ODPROJECT='ICQRBAT'
        BUILDINFONAME = 'AZMEDTECH.ICQRBAT'

        //WEBEX_URL = 'https://webexapis.com/v1/messages'  //doesnt change
        //WEBEX_ROOM = 'insert-WEBEX-space-link-here'  //FILL alpha numeric space link from webex space to post notifications to
        //APPNAME = 'PackagenameHere' //FILL Folder for Artifactory same as package name in Ocotopus deploy
        ODCHANNEL='Default'  //FILL Default or Hotfix depending on which channel you need to create release in on Octopus Deploy
        BUILD_BRANCH='refs/heads/develop' //FILL Branch you want to build usually devmain, dev, develop, or master        
        //SCAN_BRANCH='refs/heads/develop'  //FILL Branch you want to scan
        //SONAR_PROJKEY='com.cigna.SonarQube_key_here'  //FILL SonarQube project key
        //SONAR_URL = 'https://sonarqube.sys.cigna.com/projects'  //doesnt change SonarQube url for webex/email notifications 
        //CX_TEAM = 'CXTeamNameHere' //FILL checkmarx team name
        //CX_PROJECT_NAME = 'CXProjectNameHere' //FILL checkmarx project name
        //CHECKMX_URL = 'https://cigna.checkmarx.net/cxwebclient/ProjectState.aspx'  //doesnt change Checkmarx Dashboard url for webex/email notifications

        //these items found or added to jenkins credentials  team folder
        //JK_GITHUB_CRED_ID = 'github_svcPAT'  //usually doesnt change - credential set up in Jenkins for Github access - part of group G_GTHB_CIG_CMADEVOPS_ADM 
        //JK_SONARQ_CRED_ID = 'CMA_SonarTK' //usually doesnt change - enterprise credential set up in Jenkins at \CHSDEVOPS level for SonarQube token
        //JK_CX_CRED_ID = 'CMA_CHXAPIKEY'  //FILL credential set up in Jenkins for Checkmarx team token


        MSBUILD = "c:\\program files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\MsBuild\\current\\bin\\msbuild.exe"
        APP_VERSION = """${powershell (returnStdout: true,
            script: "\$(get-date -format u).Replace('-','.').Substring(0,10)+\".\$(\$ENV:BUILD_ID)\" "
            )}""".trim()
    }
    parameters {
        string(name: 'BRANCH', defaultValue: 'unknown', description: '')
        string(name: 'COMMIT', defaultValue: 'unknown', description: '')
        string(name: 'COMMITS', defaultValue: 'unknown', description: '')
        string(name: 'HTML_URL', defaultValue: 'unknown', description: '')
        string(name: 'BY', defaultValue: 'unknown', description: '')
        string(name: 'GIT_REPO_URL', defaultValue: 'unknown', description: '')
        string(name: 'GIT_URL', defaultValue: 'unknown', description: '')
    }
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'BRANCH', value: '$.ref', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown'],
                [key: 'COMMIT', value: '$.after', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown'],
                [key: 'COMMITS', value: '$.commits', expressionType: 'JSONPath', regexpFilter: '', defaultValue: ''],
                [key: 'HTML_URL', value: '$.repository.html_url', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown'],
                [key: 'BY', value: '$.pusher.name', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown'],
                [key: 'GIT_REPO_URL', value: '$.repository.url', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown'],
                [key: 'GIT_URL', value: '$.repository.clone_url', expressionType: 'JSONPath', regexpFilter: '', defaultValue: 'unknown']
            ],
            causeString: '$BY Triggered build on $BRANCH by token from $GIT_REPO_URL',
            token: 'c87a9435-c66e-420e-bfb5-a7a6a9acd7c7',
            printContributedVariables: true,
            printPostContent: true,
            silentResponse: false
        )
    }
    //Stages
    stages {
        stage('Setup') {
            steps {
                echo "********** Preparing changes for the branch ${env.BRANCH}"
                echo "The previous result was ${env.PREVIOUS_BUILD_STATUS}"

                cleanWs()
                // Sending notifications
                emailext attachLog: true,
                    body: "${env.BY} started build #${env.APP_VERSION} for ${env.BRANCH}",
                    subject: "${env.BY} started build #${env.APP_VERSION} for ${env.BRANCH}",
                    recipientProviders: [developers(), requestor()],
                    to: "${env.MAIL_GROUP}"
                // Setting displayName
                script {
                    currentBuild.displayName = "${env.APP_VERSION} on ${env.BRANCH}"
                }
                checkout changelog: false,
                    poll: false,
                    scm: [$class: 'GitSCM', branches: [[name: "${env.BRANCH}"]],
                        browser: [$class: 'GithubWeb', 
                            repoUrl: "${env.GIT_URL}"],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [],
                            gitTool: 'Default_Win',
                            submoduleCfg: [],
                            userRemoteConfigs: [[credentialsId: 'github_svcPAT', url: "${env.GIT_URL}"]]
                        ]                                              
            }
          }
        stage('Build') {
            steps {
              echo "********** Building changes from the branch ${env.BRANCH} for version ${env.APP_VERSION}"
                dir('ICQRBAT_Database') {
                    bat "\"${MSBUILD}\" \"ICQRBAT_Database.sqlproj\" /nologo /nr:false /p:platform=\"Any CPU\" /p:Version=%APP_VERSION% /p:configuration=\"Release\" /t:clean;restore;rebuild"
                    bat "nuget pack ICQRBAT_Database.nuspec -version %APP_VERSION% -OutputDirectory .. -Properties Configuration=Release"
                }        
                dir ('ICQRBAT') {
                    // pack the nupkg file
                    bat "nuget pack ICQRBAT.nuspec -version %APP_VERSION% -OutputDirectory .. -Properties Configuration=Release"
                }
            }
        }
        stage('Publish') {
            
            environment {
                DESTINATION = 'Artifactory' //'Artifactory' or 'OctopusDeploy'
                ARTIFACTORYURL = 'https://cigna.jfrog.io/artifactory/api/nuget/v3/cigna-nuget-lower/Cigna/GBS/Medicare'
                ARTIFACTORYGENERICREPONAME = 'cigna-nuget-lower'
                OCTOPUS_CLI_SERVER = 'https://octopusdeploy.sys.cigna.com/'
                CHANNEL = 'Default'
                CRED_ARTIFACTORY = 'deployer-chs-devops'
                
                CRED_OCTOPUS = credentials('ODApiKey')
                OCTOPUS_SERVER = 'https://octopusdeploy.sys.cigna.com/'
                NUGET_APP_VERSION = """${powershell (returnStdout: true,
                    script: "\$(\$ENV:APP_VERSION).Replace('.0','.')"
                    )}""".trim()
            }
            steps {
                    echo "********** Entering the Publish stage ****************"
                    withCredentials([usernamePassword(credentialsId: "${env.CRED_ARTIFACTORY}",
                    passwordVariable: 'deployerCred',
                    usernameVariable: 'deployerId')]) {
                    powershell """
                        \$ErrorActionPreference = "Stop"
                        Write-Host ""
                        \$ArtifactoryDeployLocation = "${env.ARTIFACTORYURL}/${env.BUILDINFONAME}/"
                        Write-Host "Artifactory Upload Target \$ArtifactoryDeployLocation"
                        dotnet nuget push "*.nupkg" --api-key "${deployerId}:${deployerCred}" --source "\$ArtifactoryDeployLocation" --skip-duplicate
                        #dotnet nuget push "ICQRBAT_Database\\*.nupkg" --api-key "${deployerId}:${deployerCred}" --source "\$ArtifactoryDeployLocation" --skip-duplicate
                    """
                    
                 // Create release
                    //bat "octo create-release --project \"${env.ODPROJECT}\" --version \"${env.APP_VERSION}\" --channel \"${env.CHANNEL}\" --apiKey \"${CRED_OCTOPUS_PSW}\" --debug"
                    powershell """
                    octo create-release `
                        --project "${env.ODPROJECT}" `
                        --version "${env.APP_VERSION}" `
                        --server "${env.OCTOPUS_SERVER}" `
                        --channel "${env.ODCHANNEL}" `
                        --apiKey "${env.CRED_OCTOPUS_PSW}"
                """
            }
        }
    }
}

    post {
        always {
            archiveArtifacts artifacts: '**/*.nupkg', excludes: '**/SSISBuild*.nupkg'
            emailext attachLog: true,
                body: "${env.BY} started build #${env.APP_VERSION} for ${env.BRANCH} \r\nResult #${env.APP_VERSION}: ${currentBuild.currentResult}",
                subject: "${env.BY} started build #${env.APP_VERSION} for ${env.BRANCH}",
                recipientProviders: [developers(), requestor()],
                to: "${env.MAIL_GROUP}"
            }
        }
}
