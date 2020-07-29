@Library(['rivigo-jenkins-lib@zoom', 'zoom-kubernetes@master']) _

import com.rivigo.zoom.utils.GitUtils
import baremetal.jenkins.utils.DeploymentUtils

pipeline {
    // do you care about jenkins agent?
    agent none
    // tools
    tools { gradle "gradle-5.2" }
    // global configurations
    options {
        disableResume()
        buildDiscarder logRotator(artifactDaysToKeepStr: '3', artifactNumToKeepStr: '5', daysToKeepStr: '15', numToKeepStr: '35')
        disableConcurrentBuilds()
        durabilityHint('MAX_SURVIVABILITY') // do not build job in production with this mode
        skipDefaultCheckout(false)
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }
    //build parameter variables
    parameters {
        // environment (namespace)
        string(name: 'ENVIRONMENT', defaultValue: "staging", trim: true)
        // build commons
        booleanParam(name: 'BUILD_ARTIFACT', defaultValue: true)
        // zoom commons
        string(name: 'COMMONS_BRANCH', defaultValue: "develop", trim: true)
        // microservice branch
        string(name: 'BRANCH', defaultValue: "develop", trim: true)
        // junits
        booleanParam(name: 'RUN_JUNITS', defaultValue: false)
        // zoom kubernetes branch
        string(name: 'ZOOM_KUBERNETES_BRANCH', defaultValue: "master", trim: true)
        // microservice
        choice(choices: ['backend', 'communication', 'datastore', 'riconet', 'ticketing', 'wms', 'zoom-book'], name: 'MICROSERVICE')
    }
    //environment variables
    environment {
        DEBUG = "#!/bin/sh -e\n"
    }
    // actual parallel staged jenkins declarative pipeline
    stages {


        stage('initial setup') {
            agent any
            steps {
                script{
                    new DeploymentUtils().retrieveEbExtensionPath()
                }
                echo "loading environment variables..."
                load "${EB_EXTENSION_DIRECTORY}/.envvars/zoom_k8s_nx.groovy"
                script{
                    new DeploymentUtils().prepareNamespace("${ENVIRONMENT}")
                    new DeploymentUtils().setEnvironmentVariablesForBackendBuildJobs(MICROSERVICE)
                }
                echo "Current Build namespace: ${NAMESPACE}"
            }
        }

        stage('trigger downstream job') {
            agent any
            steps {
                script{
                    if(env.BUILD_WITH_COMMONS) {
                        new DeploymentUtils().triggerBackendBuildJobWithCommons(BUILD_JOB_TO_TRIGGER)
                    } else {
                        new DeploymentUtils().triggerBackendBuildJob(BUILD_JOB_TO_TRIGGER)
                    }

                }
            }
        }


    }
    post {
        success {
            echo "\n\n================================================================================================="
            echo "Current Build namespace: ${NAMESPACE} and zoom-k8s-nutanix-host: ${ZOOM_K8S_NUTANIX_HOST} "
            echo "Please find your deployment : ${ZOOM_K8S_NUTANIX_HOST} "
            echo "=================================================================================================\n\n"
        }
    }
}
