#!/usr/bin/env groovy

//@Library(['github.com/emichaf/myshared@master']) _

// github
//@Library('myshared@master') _

//gerrit
@Library('myshared@master') _


library identifier: 'temp-lib@master', retriever: modernSCM(
[$class: 'GitSCMSource',
    remote: 'https://emichaf@gerrit.ericsson.se/a/eiffel/eiffel2/eiffel-ci-cd-cr/eiffel-pipeline-shared',
    excludes: '',
    includes: '*',
    rawRefSpecs: '+refs/heads/*:refs/remotes/origin/* +refs/tags/*:refs/remotes/origin/tags/* +refs/changes/*:refs/remotes/changes/*'
])

Java_K8S_CI_Pipeline_Travisfile_Test{
     ARM_URL = "https://eiffel.lmera.ericsson.se/nexus/content/repositories/releases/emichaftest/com/ericsson/eiffel/eiffel-intelligence-frontend"
     DOCKER_HOST = "tcp://docker104-eiffel999.lmera.ericsson.se:4243"
     SOURCE_CODE_REPO = "https://github.com/eiffel-community/eiffel-intelligence-frontend.git"
     WRAPPER_REPO = "https://github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git"
     BUILD_INFO_FILE = 'build_info.yml'
     BUILD_COMMAND = "mvn install -DskipTests=true -Dmaven.javadoc.skip=true -B -V"
     SONARQUBE_HOST_URL = "https://sonarqube.lmera.ericsson.se"
     DOCKERIMAGE_BUILD_TEST = "emtrout/nind23"
     DOCKERIMAGE_DOCKER_BUILD_PUSH = "emtrout/nind23"
     JAR_WAR_EXTENSION = "war"
}
