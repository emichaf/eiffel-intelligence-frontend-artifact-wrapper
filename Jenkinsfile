#!/usr/bin/env groovy

@Library(['github.com/emichaf/myshared@master']) _

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
