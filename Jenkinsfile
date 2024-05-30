def imageName = "td-etlhot-be"
def serviceType = "be"
def repoUrl = scm.userRemoteConfigs[0].url

def failed_stage_name

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
    }
}
