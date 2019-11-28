pipeline {
    agent {
        label any
    }

    environment {

       

        DOCKER_REPO= "docker-local"

        DEBIAN_REPO = "debian-local"

    }

    stages {

        stage("Pull  Debian") {

            steps {

                dir("${WORKSPACE}/build/pkg/pool/main/p/python3.7") {

                    sh "wget http://ppa.launchpad.net/deadsnakes/ppa/ubuntu/pool/main/p/python3.7/libpython3.7-dev_3.7.5-1+xenial1_amd64.deb"

                }

            }

        }

        stage("Publish debain to artifactory") {

            steps {

                script {

                    rtBuildInfo (

                        maxBuilds: 3,

                        doNotDiscardBuilds: ["3"],

                        captureEnv: true,

                        deleteBuildArtifacts: true,

                        buildName : "${JOB_NAME}",

                        buildNumber:  "${BUILD_NUMBER}"

                    )

                    rtUpload (

                        serverId: "artifactory-aws",

                        buildName: "${JOB_NAME}",

                        buildNumber: "${BUILD_NUMBER}",

                        spec: """{

                                "files": [

                                    {

                                        "pattern": "${WORKSPACE}/build/pkg/",

                                        "target": "${DEBIAN_REPO}/",

                                        "props" : "deb.distribution=xenial;deb.component=main;deb.architecture=amd64"

                                    }

                                ]

                            }"""

                    )

                    rtPublishBuildInfo (

                        serverId: "artifactory-jfrog",

                        buildName: "${JOB_NAME}",

                        buildNumber: "${BUILD_NUMBER}"

                    )

                }

            }

        }

        stage("Pull docker images") {

            steps {

                script {

                    sh "docker pull alpine:latest"

                    sh "docker pull golang:latest"

                    sh "docker tag alpine:latest artifactory.jfrog.com/test-docker/alpine:latest"

                    sh "docker tag golang:latest artifactory.jfrog.com/test-docker/golang:latest"

                }

            }

        }

        stage("Push docker images") {

            steps {

                script {

                    def awsbuildInfo = newBuildInfo()

                    awsbuildInfo.setName("docker-${JOB_NAME}")

                    def server = Artifactory.server "artifactory-jfrog"

                    def rtDocker = Artifactory.docker server: server

                    awsbuildInfo.retention(["maxBuilds": 2,  "deleteBuildArtifacts" : true])

                    collectEnv(awsbuildInfo.getEnv())

                    awsbuildInfo.append(rtDocker.push("artifactory.jfrog.com/test-docker/alpine:latest", "${DOCKER_REPO}"))

                    awsbuildInfo.append(rtDocker.push("artifactory.jfrog.com/test-docker/golang:latest", "${DOCKER_REPO}"))

                    server.publishBuildInfo awsbuildInfo

                }

            }

        }

    }

}
