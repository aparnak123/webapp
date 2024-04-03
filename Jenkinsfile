#!/usr/bin/env groovy

// def artifactoryServer = Artifactory.server('build.ge.artifactory')
// def snapshotRepo = 'KYPNN-SNAPSHOT'
// def releaseRepo = 'KYPNN'
def mainBranch = 'main'
def developmentBranch = 'dev'
def githubOrg = 'DT-DIVE'
def productName = 'DIVE'
def dtrOrgName = 'digital-connect'
def featureBranch = (env.BRANCH_NAME != mainBranch && env.BRANCH_NAME != developmentBranch)

pipeline {
    agent none
    environment {
        COMPLIANCEENABLED = true
        DTR = credentials('dtr-functional-user')
//        ARTIFACTORY_API_KEY = credentials('build-ge-artifactory-api-key')
    }
    parameters {
        string(name: 'BUILD', defaultValue: "true", description: 'whether or not to build elasticssearch docker image')
        string(name: 'DIVE_VERSION', defaultValue: "2021_06_v1", description: 'current DIVE version to build and tag as.')
//        choice(name: 'DIVE_CONFIG_FOLDER', choices: ['config', 'config-eks'], description: 'Enter "config" for docker-swarm and "config-eks" for EKS')
        string(name: 'ELK_VERSION', defaultValue: "8.8.1", description: 'current ELK version DIVE is using.')
//        string(name: 'ARTIFACTORY_REPO', defaultValue: "KYPNN-SNAPSHOT", description: 'Artifactory repo to obtain artifacts from.')
//        string(name: 'INTERNAL_OR_EXTERNAL_BINARY', defaultValue: "external", description: 'Original source of built binary, either internal or external.')
//        string(name: 'ARTIFACTORY_BINARY', defaultValue: "elasticsearch-8.8.1-linux-x86_64.tar.gz", description: 'Name of binary artifact to obtain.')
    }
    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '10', daysToKeepStr: '', numToKeepStr: '10'))
    }
    stages {
      stage ('Build Elasticsearch Image') {
        agent { label 'dind' }
        when {
            expression { return env.BUILD == "true"; }
          }
        steps {
            script {
                echo "Building Elasticsearch Docker image"
                def imageTag = (env.BRANCH_NAME == mainBranch) ? "${env.DIVE_VERSION}-elk-v${env.ELK_VERSION}" : "${env.DIVE_VERSION}-elk-v${env.ELK_VERSION}-${env.BRANCH_NAME}"

                sh "docker login registry.gear.ge.com -u $DTR_USR -p $DTR_PSW"
/*                
                sh "docker build . \
                --build-arg ARTIFACTORY_API_KEY=${ARTIFACTORY_API_KEY} \
                --build-arg ELK_VERSION=${env.ELK_VERSION} \
                --build-arg DIVE_VERSION=${env.DIVE_VERSION} \
                --build-arg DIVE_CONFIG_FOLDER=${env.DIVE_CONFIG_FOLDER} \
                --build-arg ARTIFACTORY_REPO=${env.ARTIFACTORY_REPO} \
                --build-arg ARTIFACTORY_PATH=dive/elasticsearch/${env.INTERNAL_OR_EXTERNAL_BINARY}/ \
                --build-arg ARTIFACTORY_BINARY=${env.ARTIFACTORY_BINARY} \
                -t registry.gear.ge.com/dt-dive/dive-elasticsearch:$imageTag \
                -t registry.gear.ge.com/dt-dive/dive-elasticsearch:latest"
                sh "docker push registry.gear.ge.com/dt-dive/dive-elasticsearch:$imageTag"
*/
                sh "docker build . \
                --build-arg ELK_VERSION=${env.ELK_VERSION} \
                -t registry.gear.ge.com/digital-connect/dive-elasticsearch:$imageTag \
                -t registry.gear.ge.com/digital-connect/dive-elasticsearch:latest"
                sh "docker push registry.gear.ge.com/digital-connect/dive-elasticsearch:$imageTag"
                if (env.BRANCH_NAME == mainBranch) {
                    sh "docker push registry.gear.ge.com/digital-connect/dive-elasticsearch:latest"
                }
            }
          }
          post {
            success {
              echo 'Build stage successful'
            }
            failure {
              echo 'Build stage failed'
            }
          }
        }
    }
    post {
        success {
            script {
                echo 'The build has completed successfully. Notifying relevant parties...'
                if (!featureBranch) {
                    emailext (
                        subject: "Build Success: ${env.JOB_NAME}",
                        body: "Your pipeline: '${env.JOB_NAME} [${env.BUILD_NUMBER}]': Completed succesfully. Check console output at ${env.BUILD_URL}/console}",
                        to: "dt.dive@ge.com",
                        from: 'jenkins-robot@do-not-reply.com',
                        replyTo: 'jenkins-robot@o-not-reply.com'
                    )
                }
            }
        }
        failure {
            script {
                echo 'The build was unsuccessful. Notifying relevant parties...'
                if (!featureBranch) {
                    emailext (
                        subject: "Build Failure: ${env.JOB_NAME}",
                        body: "Your pipeline: '${env.JOB_NAME} [${env.BUILD_NUMBER}]': has an error. Check console output at ${env.BUILD_URL}/console}",
                        to: "dt.dive@ge.com",
                        from: 'jenkins-robot@do-not-reply.com',
                        replyTo: 'jenkins-robot@o-not-reply.com'
                    )
                }
            }
        }
    }
}
