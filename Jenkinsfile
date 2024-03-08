pipeline {
    libraries {
        lib('utils')
    }
    agent {
        label 'linux-perm'
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: "30"))
        disableConcurrentBuilds()
        timeout(time: 5, unit: "MINUTES")
    }
    parameters {
        string(name: "BRANCH")
    }
    stages {
        stage("env") {
            steps {
                script {
                    withCreds("github-token-pr") {
                        def prDetails = readJSON(text: httpRequest(url: 'https://api.github.com/repos/brave/brave-site-specific-scripts/pulls?head=brave:' + BRANCH, customHeaders: [[name: 'Authorization', value: 'token ' + PR_BUILDER_TOKEN]], quiet: false).content)[0]
                        env.SLACK_USERNAME = utils.getSlackUserName(prDetails?.user?.login)
                        env.SLACK_PREFIX = "[brave-site-specific-scripts-pr]\n<https://github.com/brave/brave-site-specific-scripts/pull/${prDetails.number}|PR-${prDetails.number}>\n${BRANCH} `${BUILD_NUMBER}`"
                    }
                }
            }
        }
        stage("checkout") {
            steps {
                script {
                    FAILED_STAGE = env.STAGE_NAME
                    checkout([$class: "GitSCM", branches: [[name: BRANCH]], extensions: [[$class: "WipeWorkspace"]], userRemoteConfigs: [[url: "https://github.com/brave/brave-site-specific-scripts.git"]]])
                }
            }
        }
        stage("install") {
            steps {
                script {
                    FAILED_STAGE = env.STAGE_NAME
                    sh 'npm ci'
                }
            }
        }
        stage("lint") {
            steps {
                script {
                    FAILED_STAGE = env.STAGE_NAME
                    sh 'npm run lint'
                }
            }
        }
        stage("build") {
            steps {
                script {
                    FAILED_STAGE = env.STAGE_NAME
                    sh 'npm run build'
                }
            }
        }
        stage("test") {
            steps {
                script {
                    FAILED_STAGE = env.STAGE_NAME
                    sh 'npm run test'
                }
            }
        }
    }
}

