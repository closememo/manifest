pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    agent any

    parameters {
        string(name: 'component', defaultValue: '')
        string(name: 'phase', defaultValue: 'dev')
        string(name: 'tag', defaultValue: 'latest')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    if ("${params.component}" == '') {
                        currentBuild.result = 'ABORTED'
                        error("component parameter must not be empty.")
                    }
                }
                echo "component: ${params.component}"
                echo "phase: ${params.phase}"
                echo "tag: ${params.tag}"

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'refs/heads/master']],
                    extensions: [[$class: 'CleanCheckout']],
                    userRemoteConfigs: [[credentialsId: 'jenkins-bitgadak-closememo', url: 'https://github.com/closememo/manifest.git']]
                ])
            }
        }
        stage('Change tag') {
            steps {
                script {
                    def filename = 'argocd/' + params.component + '/' + params.phase + '/deployment.yaml'
                    def deployment = readYaml file: filename
                    def imageParts = deployment.spec.template.spec.containers[0].image.split(':') as List

                    imageParts.remove(imageParts.size() - 1)
                    imageParts.add(params.tag)

                    def newImage = imageParts.join(':')
                    deployment.spec.template.spec.containers[0].image = newImage

                    writeYaml file: filename, data: deployment, overwrite: true
                }
            }
        }
        stage('Push') {
            steps {
                withCredentials([gitUsernamePassword(
                        credentialsId: 'jenkins-bitgadak-closememo',
                        gitToolName: 'git-tool')]) {
                    sh 'git config --local user.name bitgadak'
                    sh 'git config --local user.email bitgadak@gmail.com'

                    sh "git diff --quiet && git diff --staged --quiet || git commit -am \"[${params.component}] change tag: ${params.tag}\""
                    sh 'git push origin HEAD:master'
                }
            }
        }
    }
}
