node{

  // ######### NOTES & INFORMATION & WARNINGS ##############################################################################
  //
  // OBS change dir in containers not working, so fetching scm in containers is required. Stash/unstash dir() not working..
  // https://issues.jenkins-ci.org/browse/JENKINS-46636
  // https://issues.jenkins-ci.org/browse/JENKINS-33510
  // Even context (docker build -f xxxx/Dockerfile  yy/context) when copying files from host to image in Dockerfile is not working
  //  - Solution: Add jar file (to be copied in the Dockerfile) in the same dir as the Dockerfile
  //
  // ######### NOTES & INFORMATION & WARNINGS ##############################################################################


     String GIT_SHORT_COMMIT
     String GIT_LONG_COMMIT
     String GITHUB_HASH_TO_USE
     String ARM_ARTIFACT
     String ARM_ARTIFACT_PATH
     Object pom
     String ARM_URL = "https://eiffel.lmera.ericsson.se/nexus/content/repositories/releases/test/com/ericsson/eiffel-intelligence-frontend-artifact-wrapper"
     String DOCKER_HOST = "tcp://docker104-eiffel999.lmera.ericsson.se:4243"
     String WRAPPER_REPO = "https://github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git"
     String SOURCE_CODE_REPO = "https://github.com/emichaf/eiffel-intelligence-frontend.git"


 docker.withServer("$DOCKER_HOST", 'remote_docker_host') {

     /*------------------------------------------------------------------------------------------
     For inside() to work, the Docker server and the Jenkins agent must use the same filesystem,
     so that the workspace can be mounted.

     When Jenkins detects that the agent is itself running inside a Docker container, it will automatically pass
     the --volumes-from argument to the inside container, ensuring that it can share a workspace with the agent.

     ------------------------------------------------------------------------------------------*/

       stage ('GERRIT Checkout') {

              dir ('wrapper') {
                            git branch: "master", url: "$WRAPPER_REPO"
                            //git branch: "$env.BRANCH_NAME", url: "$WRAPPER_REPO"


                            // Read build info file with github hash
                            String file_name = 'build_info.yaml'
                            sh "cat $file_name"
                            def props = readYaml file: "$file_name"
                            GITHUB_HASH_TO_USE = props.commit

                            sh "echo hash -> $GITHUB_HASH_TO_USE"

                            sh "pwd"
                            sh "ls"
                            sh "ls src"
              }

        }



       stage ("GITHUB Checkout: $GITHUB_HASH_TO_USE") {

              dir ('sourcecode') {

                  checkout scm: [$class: 'GitSCM',
                          userRemoteConfigs: [[url: "$SOURCE_CODE_REPO"]],
                          branches: [[name: "$GITHUB_HASH_TO_USE"]]]

                          sh "pwd"
                          sh "ls"
                          sh "ls src"

                          GIT_SHORT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()

                          GIT_LONG_COMMIT =  sh(returnStdout: true, script: "git log --format='%H' -n 1").trim()

                          pom = readMavenPom file: 'pom.xml'

                          ARM_ARTIFACT = "${pom.artifactId}-${pom.version}.jar"

                          ARM_ARTIFACT_PATH = "${ARM_URL}/${pom.version}/${ARM_ARTIFACT}"

                          sh "pwd"
                          sh "ls"
                          sh "ls src"
              }

        }








    dir ('sourcecode') {  // workaround to change dir outside container, not working inside container execution.. yet, see issues stated on top of file!


           docker.image('emtrout/dind:latest').inside("--privileged") {


           /*
                       stage ('SonarQube Code Analysis') {

                                         //docker.image('sonarqube').withRun('-p 9000:9000 -p 9092:9092 -e "SONARQUBE_JDBC_USERNAME=sonar" -e "SONARQUBE_JDBC_PASSWORD=sonar" -e "SONARQUBE_JDBC_URL=jdbc:postgresql://localhost/sonar"') { c ->
                                         //docker.image('sonarqube').withRun('docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube') { c ->


                                                //dir ('wrapper') {
                                                        docker.image('emtrout/dind:latest').inside {

                                                              //sh 'mvn sonar:sonar -Dsonar.host.url=http://localhost:9000'

                                                              sh 'mvn sonar:sonar -Dsonar.host.url=https://sonarqube.lmera.ericsson.se'


                                                              //sh 'mvn sonar:sonar -Dsonar.host.url=http://docker104-eiffel999.lmera.ericsson.se:9000 -Dsonar.login=1c8363811fc123582a60ed4607782902e2f5ecc9'


                                                        }

                                                //}

                                         //}


                           }
           */



                stage('SonarQube Code Analysis') {
/*
                   //sh 'mvn sonar:sonar -Dsonar.host.url=https://sonarqube.lmera.ericsson.se'
                   sh 'mvn sonar:sonar -Dsonar.host.url=http://docker104-eiffel999.lmera.ericsson.se:9000 -Dsonar.login=1c8363811fc123582a60ed4607782902e2f5ecc9'
*/

                }



                stage('Compile') {

                      sh 'mvn clean package -DskipTests'

                      sh 'ls target'

                }


                stage('UnitTests & FlowTests with TestDoubles)') {
            /*
                      // OBS privileged: true for image for embedded mongodb (flapdoodle) to work
                      // and glibc in image!

                      def travis_datas = readYaml file: ".travis.yml"

                      // Execute tests (steps) in travis file, ie same file which is used in travis build (open source)
                      travis_datas.script.each { item ->
                              sh "$item"
                      };

                      sh "ls"
                      sh "ls target"

                      */
                }


                stage('Publish Artifact ARM -> JAR)') {

                       withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                              credentialsId: '8829c73e-19b0-4f77-b74c-e112bbacd4d5',
                                              usernameVariable: 'EIFFEL_NEXUS_USER',
                                              passwordVariable: 'EIFFEL_NEXUS_PASSWORD']]) {

                              sh 'ls'

                              // Upload to ARM (ex eiffel-intelligence-0.0.1-SNAPSHOT.jar)
                              sh "curl -v -u ${EIFFEL_NEXUS_USER}:${EIFFEL_NEXUS_PASSWORD} --upload-file ./target/${ARM_ARTIFACT} ${ARM_ARTIFACT_PATH}"
                      }
                }


           } // docker.image(.....

    } // dir ('sourcecode') {




    dir ('wrapper') {  // workaround to change dir outside container, not working inside container execution.. yet, see issues stated on top of file!

       docker.image('emtrout/dind:latest').inside {

           stage('Build and Push Docker Image to Registry') {


                               def exists = fileExists '/src/main/docker/app.jar'
                               if (exists) {
                                   sh "rm /src/main/docker/app.jar"
                               }


                               withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                              credentialsId: '8829c73e-19b0-4f77-b74c-e112bbacd4d5',
                                              usernameVariable: 'EIFFEL_NEXUS_USER',
                                              passwordVariable: 'EIFFEL_NEXUS_PASSWORD']]) {



                                   // Fetch Artifact (jar) from ARM
                                   sh "curl -X GET -u ${EIFFEL_NEXUS_USER}:${EIFFEL_NEXUS_PASSWORD} ${ARM_ARTIFACT_PATH} -o src/main/docker/app.jar"

                                   sh "ls src/main/docker/"

                                }


                                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                            credentialsId: '7b05ac28-c1ae-4249-a0c6-7c54c74e3b67',
                                            usernameVariable: 'DOCKER_HUB_USER',
                                            passwordVariable: 'DOCKER_HUB_PASSWORD']]) {

                                   sh "ls src/main/docker/"

                                   sh "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}"

                                   sh "docker build --no-cache=true -t ${DOCKER_HUB_USER}/${pom.artifactId}:latest -f src/main/docker/Dockerfile src/main/docker/"

                                   sh "docker push ${DOCKER_HUB_USER}/${pom.artifactId}:latest"

                                   sh "docker build --no-cache=true -t ${DOCKER_HUB_USER}/${pom.artifactId}:${GIT_SHORT_COMMIT} -f src/main/docker/Dockerfile src/main/docker/"

                                   sh "docker push ${DOCKER_HUB_USER}/${pom.artifactId}:${GIT_SHORT_COMMIT}"

                                   sh "docker logout"

                                   }

           } // stage('..

       } // docker.image('....

    } // dir ('wrapper') {



 } //  docker.withServer(...

} // node
