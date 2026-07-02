pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
        cron('H/5 * * * *')
    }

    tools {
        maven 'maven_lts'
    }

    environment {
        EXTERNAL_REPO = 'https://github.com/rzuazua/demos-devops.git'
        EXTERNAL_BRANCH = 'main'
        LAST_COMMIT_FILE = '.last-external-commit'
        RUN_EXTERNAL_BUILD = 'false'
    }

    stages {
        stage('Check external repo') {
            steps {
                script {
                    def remoteCommit = sh(
                        script: "git ls-remote ${env.EXTERNAL_REPO} refs/heads/${env.EXTERNAL_BRANCH} | cut -f1",
                        returnStdout: true
                    ).trim()

                    def savedCommit = fileExists(env.LAST_COMMIT_FILE)
                        ? readFile(env.LAST_COMMIT_FILE).trim()
                        : ''

                    echo "External remote commit: ${remoteCommit}"
                    echo "Last processed commit: ${savedCommit}"

                    if (remoteCommit != savedCommit) {
                        env.RUN_EXTERNAL_BUILD = 'true'
                        writeFile file: env.LAST_COMMIT_FILE, text: remoteCommit
                        echo 'External repository changed. Pipeline will continue.'
                    } else {
                        currentBuild.description = "No changes in external repo"
                        echo 'No changes in external repository. Pipeline will stop.'
                    }
                }
            }
        }

        stage('Prepara') {
            when {
                expression { env.RUN_EXTERNAL_BUILD == 'true' }
            }
            steps {
                echo "BRANCH_NAME: ${env.BRANCH_NAME}"
                git branch: env.EXTERNAL_BRANCH, url: env.EXTERNAL_REPO
            }
        }

        stage('Compile') {
            when {
                expression { env.RUN_EXTERNAL_BUILD == 'true' }
            }
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit test') {
            when {
                expression { env.RUN_EXTERNAL_BUILD == 'true' }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build') {
            when {
                expression { env.RUN_EXTERNAL_BUILD == 'true' }
            }
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy') {
            when {
                expression { env.RUN_EXTERNAL_BUILD == 'true' }
            }
            steps {
                sh 'mvn install -DskipTests'
            }
        }
    }

    post {
        success {
            script {
                if (env.RUN_EXTERNAL_BUILD == 'true') {
                    sh 'cp target/*.jar .'
                    archiveArtifacts artifacts: '*.jar', fingerprint: true, allowEmptyArchive: false
                }
            }
        }
    }
}