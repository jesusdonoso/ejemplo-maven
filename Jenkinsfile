import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    stages {
        stage("Paso 1: Download and checkout"){
            steps {
               checkout(
                        [$class: 'GitSCM',
                        branches: [[name: "test-sonar-jenkisfile" ]],
                        userRemoteConfigs: [[url: 'https://github.com/jesusdonoso/ejemplo-maven.git']]])
            }
        }
        stage("SonarQube analysis") {
            steps {
            withSonarQubeEnv(credentialsId: 'ad0a3e01df936235628b32fc893d58dd4f7b89b9', installationName: 'sonarqube')
            }
        }
        stage("Paso 3: Compliar"){
            steps {
                script {
                sh "echo 'Compile Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 4: Testear"){
            steps {
                script {
                sh "echo 'Test Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 5: Build .Jar"){
            steps {
                script {
                sh "echo 'Build .Jar!'"
                // Run Maven on a Unix agent.
                sh "mvn clean package -e"
                }
            }
        }
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}