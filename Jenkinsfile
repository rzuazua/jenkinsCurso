pipeline {
    agent any
    triggers { // Sondear repositorio a intervalos regulares
        pollSCM('* * * * *')
    }
    tools {
      maven 'maven_lts'
    }
    stages {
         stage("Prepara") {
            steps {
                echo "BRANCH_NAME: ${env.BRANCH_NAME}"
                git 'https://github.com/jmagit/demos-devops.git'
            }
        }
        stage("Compile") {
            steps {
                sh "mvn compile"
            }
        }
        stage("Unit test") {
            steps {
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        // stage("SonarQube Analysis") {
        //     steps {
        //         withSonarQubeEnv('SonarQubeDockerServer') {
        //             sh 'mvn clean verify sonar:sonar'
        //         }
        //         timeout(2) { // time: 2 unit: 'MINUTES'
        //           // In case of SonarQube failure or direct timeout exceed, stop Pipeline
        //           waitForQualityGate abortPipeline: waitForQualityGate().status != 'OK'
        //         }
        //     }
        // }
        stage("Build") {
            steps {
                sh "mvn package"
            }
        }
        stage("Deploy") {
            steps {
                sh "mvn install -DskipTests"
            }
        }
    }
}
