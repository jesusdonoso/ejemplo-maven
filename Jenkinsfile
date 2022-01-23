import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        NEXUS_USER         = credentials('user-nexus')
        NEXUS_PASSWORD     = credentials('pass-nexus')
    }
    stages {
        stage("Paso 0: Download and checkout"){
            steps {
                checkout(
                    [$class: 'GitSCM',
                    //Acá reemplazar por el nonbre de branch
                    branches: [[name: "feature-nexus" ]],
                    //Acá reemplazar por su propio repositorio
                    userRemoteConfigs: [[url: 'https://github.com/jesusdonoso/ejemplo-maven.git']]])
            }
        }
        stage("Paso 1: Compliar"){
            steps {
                script {
                sh "echo 'Compile Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 2: Testear"){
            steps {
                script {
                sh "echo 'Test Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 3: Build .Jar"){
            steps {
                script {
                sh "echo 'Build .Jar!'"
                // Run Maven on a Unix agent.
                sh "mvn clean package -e"
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage("Paso 4: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=new-feature-sonarqube'
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    //archiveArtifacts artifacts:'build/*.jar'
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'devops-usach-nexus',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: '.jar',
                                    filePath: '/var/jenkins_home/workspace/feature-nexus/build/DevOpsUsach2020-0.0.1.jar']
                                ],
                        mavenCoordinate: [
                            artifactId: 'DevOpsUsach2020',
                            groupId: 'com.devopsusach2020',
                            packaging: 'jar',
                            version: '0.0.1']
                        ]
                    ]
                }
            }
        }
        stage("Paso 5: Levantar Springboot APP"){
            steps {
                sh 'mvn spring-boot:run &'
            }
        }
        stage("Paso 6: Dormir(Esperar 100sg) "){
            steps {
                sh 'sleep 100'
            }
        }
        stage("Paso 7: Test Alive Service - Testing Application!"){
            steps {
                sh 'curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"'
            }
        }
         stage("Paso 7: Download: Nexus"){
            steps {
                //http://nexus:10003/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.2/DevOpsUsach2020-0.0.2.jar
                sh ' curl -X GET -u $NEXUS_USER:$NEXUS_PASSWORD "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1..jar" -O'
            }
        }
        stage("Paso 8: Levantar Springboot APP"){
            steps {
                sh 'nohup bash java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
            }
        }
        stage("Paso 9: Dormir(Esperar 100sg) "){
            steps {
               sh "sleep 100 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
            }
        }
        stage("Paso 10: Subir nueva Version"){
            steps {
                //archiveArtifacts artifacts:'build/*.jar'
                nexusPublisher nexusInstanceId: 'nexus',
                    nexusRepositoryId: 'devops-usach-nexus',
                    packages: [
                        [$class: 'MavenPackage',
                            mavenAssetList: [
                                [classifier: '',
                                extension: '.jar',
                                filePath: '/var/jenkins_home/workspace/feature-nexus/build/DevOpsUsach2020-0.0.1.jar']
                            ],
                    mavenCoordinate: [
                        artifactId: 'DevOpsUsach2020',
                        groupId: 'com.devopsusach2020',
                        packaging: 'jar',
                        version: '1.0.0']
                    ]
                ]
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
