#!groovy
env.RELEASE_COMMIT = "1";
env.VERSION_NAME = "";
env.REPOSITORY_NAME = "CscTrackerStarter"
env.LIBRARY_NAME = "CscTrackerStarter-starter"

pipeline {
    agent none
    stages {
        stage('CheckBranch') {
            agent any
            steps {
                script {
                    result = sh(script: "git log -1 | grep 'Triggered Build'", returnStatus: true)
                    echo 'result ' + result
                    env.RELEASE_COMMIT = result == 0 ? '0' : '1'
                }
            }
        }
        stage('Build') {
            agent any
            tools {
                maven 'M3'
            }
            when {
                expression { env.RELEASE_COMMIT != '0' }
            }
            steps {
                sh 'mvn versions:set versions:commit -DnewVersion=TEMP'
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Gerar versão') {
            agent any
            tools {
                maven 'M3'
            }
            when {
                expression { env.RELEASE_COMMIT != '0' }
            }
            steps {
                script {
                    echo 'RELEASE_COMMIT ' + env.RELEASE_COMMIT
                    if (env.BRANCH_NAME == 'master') {
                        echo 'Master'
                        VERSION = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yy"}.${BUILD_WEEK,XX}.${BUILDS_THIS_WEEK,XXX}')
                        sh 'mvn versions:set versions:commit -DnewVersion=RELEASE'
                        sh 'mvn clean install package -DskipTests'
                    } else {
                        echo 'Dev'
                        VERSION = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY,XX}.${BUILD_NUMBER,XXXXX}')
                        VERSION = VERSION + '-SNAPSHOT'
                    }

                    withCredentials([usernamePassword(credentialsId: 'gitHub', passwordVariable: 'password', usernameVariable: 'user')]) {
                        script {
                            echo "Creating a new tag"
                            sh 'git pull https://krlsedu:${password}@github.com/krlsedu/' + env.REPOSITORY_NAME + '.git HEAD:' + env.BRANCH_NAME
                            sh 'mvn versions:set versions:commit -DnewVersion=' + VERSION
                            sh 'mvn clean install package -DskipTests'
                            if (env.BRANCH_NAME == 'master') {
                                sh "git add ."
                                sh "git config --global user.email 'krlsedu@gmail.com'"
                                sh "git config --global user.name 'Carlos Eduardo Duarte Schwalm'"
                                sh "git commit -m 'Triggered Build: " + VERSION + "'"
                                sh 'git push https://krlsedu:${password}@github.com/krlsedu/' + env.REPOSITORY_NAME + '.git HEAD:' + env.BRANCH_NAME
                            }
                        }
                    }
                    env.VERSION_NAME = VERSION
                }
            }
        }


        stage('Update dos serviços dependentes') {
            agent any
            when {
                expression { env.RELEASE_COMMIT != '0' }
            }
            steps {
                script {
                    for (int i = 60; i >= 0; i--) {
                        println "Waiting... ${i} seconds remaining."
                        sleep(time: 1, unit: 'SECONDS')
                    }
                    withCredentials([string(credentialsId: 'csctracker_token', variable: 'token_csctracker')]) {
                        def xCorrelationId = 'update-lib_' + env.LIBRARY_NAME + '-' + env.VERSION_NAME
                        def url = 'https://redirect.loclx.io/updater/' + env.LIBRARY_NAME + '/' + env.VERSION_NAME
                        println 'url ' + url
                        println 'xCorrelationId ' + xCorrelationId
                        def response = httpRequest acceptType: 'APPLICATION_JSON',
                                contentType: 'APPLICATION_JSON',
                                httpMode: 'POST', quiet: true,
                                requestBody: '''{}''',
                                customHeaders: [
                                        [name: 'authorization', value: 'Bearer ' + env.token_csctracker],
                                        [name: 'x-correlation-id', value: xCorrelationId]
                                ],
                                url: '' + url
                        println 'Response: ' + response.content
                    }
                }
            }
        }
    }
}
