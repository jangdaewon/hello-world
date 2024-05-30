def imageName = "td-etlhot-be"
def serviceType = "be"
def repoUrl = scm.userRemoteConfigs[0].url
def changelog = ""
def jobNameSyncConfig = 'sync-service-config'
def registry_name = "misf-nexus.internal.synopsys.com:5002/repository/docker-pcs"

def failed_stage_name

def sendmail(stage_name) {
    emailext(
            subject: "Job '${env.JOB_NAME} - ${env.GIT_COMMIT}' Failed at ${stage_name}: ",
            body: '${FILE, path="/remote/skyfall/build-central/scripts/template.html"}',
            compressLog: true,
            attachLog: true,
            mimeType: 'text/html',
            recipientProviders: [[$class: 'DevelopersRecipientProvider']],
            to: "woojin@synopsys.com;joykim@synopsys.com;bangera@synopsys.com;hdesai@synopsys.com;amlenduc@synopsys.com;ybruce@synopsys.com;mrugansh@synopsys.com",
            from: "skyfall@synopsys.com"

    )
}

pipeline {
    agent {
        node {
            label 'mi-build'
            customWorkspace '/remote/skyfall/build-central/workspace/MI/td-etlhot-be'
        }
    }
    stages {
        stage('Build source code') {
            steps {
                script {
                    try {
                        sh "mvn clean package"
                    }
                    catch (err) {
                        env.BUILD_STATUS = 'FAILURE'
                        failed_stage_name = env.STAGE_NAME
                        throw err
                    }
                }
            }
        }
        stage('Synchronize configuration on docker-compose') {
            steps {
                script {
                    // prepare changelog message
                    def changeLogSets = currentBuild.changeSets
                    changelog = "Changelog"
                    for (int i = 0; i < changeLogSets.size(); i++) {
                        def entries = changeLogSets[i].items
                        for (int j = 0; j < entries.length; j++) {
                            def entry = entries[j]
                            changelog += "\n" + "\t- Time: ${new Date(entry.timestamp)}, Commit Hash: ${entry.commitId}, Author: ${entry.author}, Message: ${entry.msg}"
                        }
                    }
                }
                // trigger automation job with parameters
                build job: jobNameSyncConfig,
                        parameters: [
                                string(name: 'imageName', value: "${imageName}"),
                                string(name: 'serviceType', value: "${serviceType}"),
                                string(name: 'repoUrl', value: "${repoUrl}"),
                                string(name: 'changelog', value: "${changelog}")
                        ]
            }
        }
    }

    post {
        success {
            emailext(
                    subject: '$DEFAULT_SUBJECT',
                    body: '${FILE, path="/remote/skyfall/build-central/scripts/template.html"}',
                    mimeType: 'text/html',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                    to: "joykim@synopsys.com;bangera@synopsys.com;hdesai@synopsys.com;amlenduc@synopsys.com;ybruce@synopsys.com;mrugansh@synopsys.com",
                    from: "skyfall@synopsys.com"

            )
        }
        failure {
            script {
                sendmail(failed_stage_name)
            }
        }
    }
}
