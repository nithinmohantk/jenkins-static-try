properties([pipelineTriggers([githubPush()])])

pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e *.html'
            }
        }
        stage('Deploy to AWS') {
            steps {
                withAWS(region:'eu-west-1',credentials: 'aws-static') {
                    timeout(time: 3, unit: 'MINUTES') {
                        retry(5) {
                            echo '<<<<Uploading content to S3 with AWS Credentials>>>>'
                            s3Upload(pathStyleAccessEnabled:true, payloadSigningEnabled: true, file:'index.html', bucket:'mansong-jenkins-udacity')
                        }
                    }
                }
            }
        }
        stage('Post Deploy Test') {
            steps {
                echo '<<<<Testing Deployment>>>>'
                script {
                    String webSite = 'https://udacity-thingx-jenkins.s3.eu-west-1.amazonaws.com/index.html'
                    def returnCode = sh(returnStdout: true, script: "curl -sLI -o /dev/null -w '%{http_code}' ${webSite}").trim() as Integer
                    if (returnCode != 200) {
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
    /* clean working directory */
    post {
       always {
           deleteDir()
       }
    }
}
