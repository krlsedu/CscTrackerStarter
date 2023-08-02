#!groovy
env.RELEASE_COMMIT = "1";

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
                        VERSION = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY}.${BUILD_NUMBER}')
                        VERSION = VERSION + '-SNAPSHOT'
                    }

                    echo "Creating a new tag"
                    sh 'git pull origin master'
                    sh 'mvn versions:set versions:commit -DnewVersion=' + VERSION
                    sh 'mvn clean install package -DskipTests'

                    withCredentials([usernamePassword(credentialsId: 'gitHub', passwordVariable: 'password', usernameVariable: 'user')]) {
                        script {
                            sh "git add ."
                            sh "git config --global user.email 'krlsedu@gmail.com'"
                            sh "git config --global user.name 'Carlos Eduardo Duarte Schwalm'"
                            sh "git commit -m 'Triggered Build: " + VERSION + "'"
                            sh 'git push https://krlsedu:${password}@github.com/krlsedu/CscTrackerStarter.git HEAD:' + env.BRANCH_NAME
                        }
                    }
                }
            }
        }
    }
}
