pipeline {
    agent {
        label 'JKS-Agent01'
    }

    stages {
        stage("Build and test image") {
            steps {
                script {
                    // Use commit tag if it has been tagged
                    def tag = sh(returnStdout: true, script: "git tag --contains").trim()
                    if (tag == "") {
                        tag = "${BRANCH_NAME == 'master' ? 'latest' : BRANCH_NAME}"
                    }
                    withDockerRegistry(credentialsId: 'dockerhubebe', toolName: 'docker') {
                        def image = docker.build("moamensaleh/testing-image", "--build-arg 'BUILDKIT_INLINE_CACHE=1' --cache-from moamensaleh/testing-image:$tag --cache-from moamensaleh/testing-image:latest .")
                        image.inside("--volume /etc/passwd:/etc/passwd:ro") {
                            //sh label: "Test anchore-cli", script: "anchore-cli --version"
                            sh label: "Test curl", script: "curl --version"
                            sh label: "Test cyclonedx", script: "cyclonedx-py --help"
                            sh label: "Test detect-secrets", script: "detect-secrets --version"
                            sh label: "Test nikto.pl", script: "nikto.pl -Version"
                            //sh label: "Test for outdated global npm packages", script: "npm outdated --global"
                            sh label: "Test sonar-scanner", script: "sonar-scanner --version"
                            //sh label: "Test trufflehog", script: "trufflehog --help"
                        }
                    }
                }
            }
        }

        stage("Push to registry") {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: "git tag --contains").trim()
                    if (tag == "") {
                        tag = "${BRANCH_NAME == 'master' ? 'latest' : BRANCH_NAME}"
                    }
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker tag moamensaleh/testing-image moamensaleh/testing-image:$tag"
                        sh "docker push moamensaleh/testing-image"
                    }
                }
            }
        }
    }
}
