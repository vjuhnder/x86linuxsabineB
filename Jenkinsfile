#!groovy
@Library('libs@default')
/* import shared library */
@Library('jenkins-shared-library')_
import com.uhnder.helpers.*
import com.uhnder.pipelines.*
import com.uhnder.shared.*
import com.uhnder.stages.*
import com.uhnder.steps.*

def SRS_REVISION_ID
def SRA_REVISION_ID
def RHAL_EXP_REVISION_ID
def lastCommitUser
def slackData = []
    
properties([
    parameters([
        string( defaultValue: 'default',
                description: 'Specify a changeset (hash) to checkout',
                name: 'SRS_CHANGESET'),
    ]),
    buildDiscarder(logRotator(artifactDaysToKeepStr: '3', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: ''))
])

try
{
    def srs_repo  = new RepoUpdateStep(this, "system-radar-software", "https://bitbucket.org/uhnder/", params.SRS_CHANGESET)  
    def path
    
    node('master')
    {
        path = sh (
            script: 'python2.7 ../'+env.JOB_NAME+'@script/scripts/path.py',
            returnStdout: true
        ).trim()
    }
    echo path
    def p = new Pipeline(this, 40)
    p << new Stage('RepoUpdateStep', this)
        .addStep(srs_repo)
    SRS_REVISION_ID = srs_repo.get_rev_id()
 /*   p << new Stage('Building Sabine', this)
        .addStep(new BuildSrsStep(this, SRS_REVISION_ID, path,"x86_linux", "sabineB", "",""))*/
    p << new Stage('Arhive bins', this)
        .addStep(new ArchiveSRSBinsStep(this, SRS_REVISION_ID, path,"x86_linux", "sabineB", ""))

    p.execute()
 
    srs_repo = null
    sra_repo = null 
}
catch (e)
{
    echo 'The Pipeline failed :('
    throw e
}
finally
{
    stage('Build Naming')
    {
        node('master')
        {
            currentBuild.displayName = "#${BUILD_NUMBER}: SRS(${SRS_REVISION_ID})"
                        
            echo currentBuild.currentResult
            echo env.SRS_REVISION_ID
            /*Get environment variables*/
            def fields = env.getEnvironment()
                   fields.each {
                        key, value -> println("${key} = ${value}");
                    }
 
                    println(env.PATH)
            dir('system-radar-software')
            {
                lastCommitUser = sh (
                    returnStdout: true, 
                    script: "hg parent | grep user "
                    ).trim()
                echo lastCommitUser                
            }
            
            slackData.add(0, currentBuild.currentResult)
            slackData.add(1, lastCommitUser)
            /* Use slackNotifier.groovy from shared library and provide current build result as parameter */   
            slackNotifier(slackData as String[])
        }
    }
}
