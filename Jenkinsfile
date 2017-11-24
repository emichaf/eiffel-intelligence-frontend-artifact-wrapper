podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'maven', image: 'emtrout/dind:latest', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {
    node('mypod') {



        String GIT_SHORT_COMMIT


            stage ('GIT Checkout EI FrontEnd SC') {

                dir ('foo') {
                                git branch: "master", url: 'https://github.com/Ericsson/eiffel-intelligence-frontend.git'
                            }


            }




        stage('Maven Build EI FrontEnd SC') {
                    container('maven') {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                credentialsId: 'e7de4146-4a59-4406-916e-d10506cfaeb8',
                                usernameVariable: 'DOCKER_HUB_USER',
                                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {

                     sh "cd foo"
                     sh "pwd"

                     //sh "mvn clean package -DskipTests"

                     }

                    }
                }



        stage ('GIT Checkout EI Wrapper') {
            git branch: "master", url: 'https://github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git'

            GIT_SHORT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()


        }

        stage('Maven Build') {
            container('maven') {
            withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'e7de4146-4a59-4406-916e-d10506cfaeb8',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {

               sh "pwd"
               sh "cd foo"
               sh "pwd"

             //sh "mvn clean package -DskipTests"

             }

            }
        }


        stage ('Test') {
        container('maven') {
              parallel (
                'Test Server' : {
                  sh 'ls'
                },
                'Test Sample Client' : {
                  sh 'ls'
                }
              )
            }
        }



        stage('Build and Push Docker Image') {
            container('docker') {
            withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'e7de4146-4a59-4406-916e-d10506cfaeb8',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {

               pom = readMavenPom file: 'pom.xml'


               sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}"

               sh "docker build --no-cache=true -t ${env.DOCKER_HUB_USER}/${pom.artifactId}:latest -f src/main/docker/Dockerfile src/main/docker/"

               sh "docker push ${env.DOCKER_HUB_USER}/${pom.artifactId}:latest"

               sh "docker build --no-cache=true -t ${env.DOCKER_HUB_USER}/${pom.artifactId}:${GIT_SHORT_COMMIT} -f src/main/docker/Dockerfile src/main/docker/"

               sh "docker push ${env.DOCKER_HUB_USER}/${pom.artifactId}:${GIT_SHORT_COMMIT}"

               sh "docker logout"

               }
            }
        }



        stage('Deploy to K8S Stage') {
                    container('kubectl') {
                       withCredentials([[
                        $class: 'FileBinding',
                        credentialsId: '77bd756d-382a-4704-bb9c-9d52f023ac4d',
                        variable: 'KUBECONFIG'
                    ]]){


                         sh("kubectl --kubeconfig=$KUBECONFIG create -f ./target/classes/META-INF/fabric8/kubernetes.yml")
                         sh("kubectl --kubeconfig=$KUBECONFIG get pods")

                       }
                    }
                }



        stage ('Integration Test') {
                container('maven') {
                      parallel (
                        'Test Server' : {
                          sh 'ls'
                        },
                        'Test Sample Client' : {
                          sh 'ls'
                        }
                      )
                    }
                }



    }
}