// This Jenkinsfile uses the declarative syntax. If you need help, check:
// Overview and structure: https://jenkins.io/doc/book/pipeline/syntax/
// For plugins and steps:  https://jenkins.io/doc/pipeline/steps/
// For Github integration: https://github.com/jenkinsci/pipeline-github-plugin
// For credentials:        https://jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials
// For credential IDs:     https://ci.ts.sv/credentials/store/system/domain/_/
// Docker:                 https://jenkins.io/doc/book/pipeline/docker/
// Custom commands:        https://github.com/Tradeshift/jenkinsfile-common/tree/master/vars
// Environment variables:  env.VARIABLE_NAME

pipeline {
    agent any // Or you could make the whole job or certain stages run inside docker
    triggers {
        issueCommentTrigger('^(retest|docker push)$')
    }
    options {
        ansiColor('xterm')
        timestamps()
        timeout(time: 60, unit: 'MINUTES') // Kill the job if it gets stuck for too long
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '50'))
    }
    // For Java people
    tools {
        jdk 'openjdk8-jdk'
        maven 'apache-maven-3.6.0'
    }
    environment {
        P12_PASSWORD = credentials 'client-cert-password'
        MAVEN_OPTS = "-Djavax.net.ssl.keyStore=/var/lib/jenkins/.m2/certs/jenkins.p12 \
                      -Djavax.net.ssl.keyStoreType=pkcs12 \
                      -Djavax.net.ssl.keyStorePassword=$P12_PASSWORD"
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }
        stage('Compile') {
            steps {
               sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Publish') {
            when {
                anyOf {
                    branch 'master'
                    expression { getTriggerText() == 'push' } // Push PRs docker image only when requested
                }
            }
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Sonarqube') {
            when {
                anyOf {
                    branch 'master' // Only run Sonarqube on master...
                    changeRequest() // ... and PRs
                }
            }
            steps {
                sonarqube()
            }
        }
    }
}

