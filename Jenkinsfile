import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any

     environment {
        NEXUS_USER = credentials('user-nexus')
        NEXUS_PASS = credentials('password-nexus')
    }

    stages {        
                 
        stage("Paso 1: Compilar"){
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
        stage("Paso 3: build .jar"){
            steps {
                script {
                sh "echo 'build .jar!'"
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

        stage("Paso 4: Análisis SonarQube y subir a Nexus el artefacto"){
            steps {
                 withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=github-sonar'
                } 
                //sh "echo 'paso 4'"
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
                                    extension: '',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar']
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

        
         stage('Paso 5: Bajar Nexus Stage') {
            steps {
                sh 'curl -X GET -u $NEXUS_USER:$NEXUS_PASS http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar -O'
            }
        }                
   

        stage("Paso 6: Levantar Springboot APP"){
            steps {
                // sh 'mvn spring-boot:run &'
                sh 'nohup bash java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
            }
        }
        stage("Paso 7: Dormir(Esperar 20sg) "){
            steps {
                sh 'sleep 20'
            }
        }

        stage("Paso 8: Test Alive Service - Testing Application!"){
            steps {
                sh 'curl -X GET "http://nexus:8081/rest/mscovid/test?msg=testing"'
            }
        }


        stage("Paso 9: Subir nueva Version"){
            steps {
                //archiveArtifacts artifacts:'build/*.jar'
                nexusPublisher nexusInstanceId: 'nexus',
                    nexusRepositoryId: 'devops-usach-nexus',
                    packages: [
                        [$class: 'MavenPackage',
                            mavenAssetList: [
                                [classifier: '',
                                extension: '',
                                filePath: 'DevOpsUsach2020-0.0.1.jar']
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