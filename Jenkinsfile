#!groovy
node('linux') {
    slackSend baseUrl: 'https://maximusworkspace.slack.com/services/hooks/jenkins-ci/', channel: 'ssaYTTW', color: 'good', message: 'Pipeline started', tokenCredentialId: 'jenkins-slack-integration'
}

pipeline {

    agent {
		node('linuxSecondary')
	}
	
    environment {
        APP_NAME = 'ssaYTTW'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        REPOURL = 'http://svn.maximus.com/svn/ssaYTTW/lods/dev_1.5.0/'
        SBT_OPTS='-Xmx1024m -Xms512m'
        JAVA_OPTS='-Xmx1024m -Xms512m'
		JAVA_HOME='/opt/tools/java/jdk1.8.0_172'
    }
	
    options {
        timestamps()
        //retry(3)
        timeout time:10, unit:'MINUTES'
    }
	
    parameters {
		string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }

    stages {
        stage("Initialize") {
            steps {
                script {
                   // notifyBuild('STARTED')
					echo "${params.Greeting} World!"
                }
            }
        }
        stage('Checkout') {
            steps {
                echo 'Checkout Repo'
                checkout([$class: 'SubversionSCM', additionalCredentials: [], browser: [$class: 'CollabNetSVN', url: 'http://svn.maximus.com/svn/ssaYTTW/viewvc/'], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[credentialsId: '', depthOption: 'infinity', ignoreExternalsOption: false, local: '.', remote: 'http://svn.maximus.com/svn/ssaYTTW/lods/dev_1.5.0']], workspaceUpdater: [$class: 'CheckoutUpdater']])
            }
        }
        stage('Compile') {
            steps {
                echo 'Run gradle -x test --info'
                sh 'gradle -x test --info'
            }
        }
        stage('Analyze') {
            steps {
                tool name: 'SONAR_CAHCO', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
        }
        stage('Dependency Checker') {
            steps {
                echo 'Scan libs'
                dependencyCheckAnalyzer datadir: '', hintsFile: '', includeCsvReports: false, includeHtmlReports: false, includeJsonReports: false, includeVulnReports: false, isAutoupdateDisabled: false, outdir: '', scanpath: 'libs', skipOnScmChange: false, skipOnUpstreamChange: false, suppressionFile: '', zipExtensions: ''
            }
        }

        stage('Snapshot') {
            steps {
                echo "Create Release"
            }
        }

        stage('Deploy - DEV') {
            steps {
                echo "Deploy to DEV..."
            }
        }
        stage('Promote to - QA') {
            steps {
                echo "Deploy to QA..."
            }
        }
        stage('Deploy - Production') {
            when {
                expression {
                    params.DEPLOY_PROD == true
                }
            }
            steps {
                echo "Deploy to PROD..."
            }
        }
    }

    post {
        /*
         * These steps will run at the end of the pipeline based on the condition.
         * Post conditions run in order regardless of their place in pipeline
         * 1. always - always run
         * 2. changed - run if something changed from last run
         * 3. aborted, success, unstable or failure - depending on status
         */
        always {
            echo "I AM ALWAYS first"
            notifyBuild("${currentBuild.currentResult}")
        }
        aborted {
            echo "BUILD ABORTED"
        }
        success {
            echo "BUILD SUCCESS"
            echo "Keep Current Build If branch is master"
//            keepThisBuild()
        }
        unstable {
            echo "BUILD UNSTABLE"
        }
        failure {
            echo "BUILD FAILURE"
        }
    }
}


def getShortCommitHash() {
    return sh(returnStdout: true, script: "svn log -l 1 ").trim()
}

def getChangeAuthorName() {
    return sh(returnStdout: true, script: "svn info").trim()
}

def getChangeAuthorEmail() {
    return sh(returnStdout: true, script: "svn info").trim()
}

def getChangeSet() {
    return sh(returnStdout: true, script: 'svn diff -r HEAD').trim()
}

def getChangeLog() {
    return sh(returnStdout: true, script: "svn log ").trim()
}

def getCurrentBranch () {
    return sh (
            script: 'echo dev_4.0.0',
            returnStdout: true
    ).trim()
}


def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    def branchName = getCurrentBranch()
    def shortCommitHash = getShortCommitHash()
    def changeAuthorName = getChangeAuthorName()
    def changeAuthorEmail = getChangeAuthorEmail()
    def changeSet = getChangeSet()
    def changeLog = getChangeLog()

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'" + branchName + ", " + shortCommitHash
    def summary = "Started: Name:: ${env.JOB_NAME} \n " +
            "Build Number: ${env.BUILD_NUMBER} \n " +
            "Build URL: ${env.BUILD_URL} \n " +
            "Short Commit Hash: " + shortCommitHash + " \n " +
            "Branch Name: " + branchName + " \n " +
            "Change Author: " + changeAuthorName + " \n " +
            "Change Author Email: " + changeAuthorEmail + " \n " +
            "Change Set: " + changeSet

    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    // Send notifications
    slackSend baseUrl: 'https://maximusworkspace.slack.com/services/hooks/jenkins-ci/', channel: 'DMSbaseline', message: 'Pipeline ended', tokenCredentialId: 'jenkins-slack-integration'

    if (buildStatus == 'FAILURE') {
     //   emailext attachLog: true, body: summary, compressLog: true, recipientProviders: [brokenTestsSuspects(), brokenBuildSuspects(), culprits()], replyTo: '51599@maximus.com', subject: subject, to: '51599@maximus.com'
    }
}
